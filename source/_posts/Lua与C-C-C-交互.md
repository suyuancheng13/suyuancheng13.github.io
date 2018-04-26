---
title: 'Lua与C-C++-C#交互'
date: 2018-04-26 19:16:46
tags:
---


# C/C++/C#导入Lua浅入浅出

## 一、C函数导入Lua
因为Lua虚拟机就是C编写的，C函数导入Lua通过Lua C API就可以完成，相对还是比较简单。需要掌握的知识点就是Lua栈及Lua C API：  

+ Lua栈可以类比于计算机的内存  
+ Lua C API类似于汇编语言，Lua C API可以操纵Lua栈，通过操纵Lua栈可以达到与Lua交互的目的。[Lua C API 参考](https://cloudwu.github.io/lua53doc/manual.html)

### 1.1 实例    
`lua_tonumber `表示从Lua栈指定`index`处取参数，`lua_pushnumber`表示把number压入栈顶。`Add`函数本质就是从Lua栈上取两个参数然后再将两个参数的计算结果压入Lua栈供Lua获取。   

```  
int MyAdd(lua_State* l) {	int n = lua_tonumber(l,-1);	int n2 = lua_tonumber(l,-2);	int ret = n + n2;	lua_pushnumber(l,ret);	return 1;}
```
上面定义了一个C函数`MyAdd`，接下来要做的事情就是将C函数导入Lua，`lua_register`（还有其它相关API）就可以将C函数注册到Lua中，`luaL_dostring `执行段Lua代码。   

```int main(int argc, _TCHAR* argv[]){	lua_State *l = luaL_newstate();	luaL_openlibs(l);	lua_register(l,"MyAdd",MyAdd);
	//执行lua代码	luaL_dostring(l,"print(\"MyAdd result is:\");print(MyAdd(100,200))");   
}
	
```
  


## 二、C++导入Lua  
上文已经简单了解了C函数导入Lua的原理，C++类的方法的导入可以通过C函数Wrapper C++的方法，然后再将C函数导入Lua。但是需要注意的是怎么样在Lua访问C++的对象，一般可以通过`userdata`存储C++对象的指针，然后访问时从`userdata`中取。    
一般将C++类导入Lua里分为三步：  

+ 首先，创建一个`userdata` 存放 C++ 对象指针    
+ 然后给 `userdata` 添加元表
+ 用 `__index` 元方法映射 C++ 中的方法。    

### 2.1 实例

#### 1、 准备将要导入Lua的自定义C++ `Account`类

```
class Account {public: 	Account(double balance){ m_balance = balance;}	void deposit(double amount){m_balance += amount;}	void withdraw(double amount){m_balance -= amount;}	double balance(){return m_balance;}private:	double m_balance;};```
####2、 “构造”函数
通过`lua_newuserdata `创建一个`userdata`，并将`Account`对象的指针存储于`userdata`。然后，通过`lua_setmetatable `给`userdata`设置元表（元表会在注册时创建，见下文）。    
至于为什么需要给`userdata`关联元表，因为`userdata`只是一个指针而已没有任何方法，所以需要设置一个元表（元表也是一个table）。 lua的查找路径：

1. 在当前的表中查找，如果找到，返回该元素，找不到则继续2。
2. 判断该表是否有元表，如果没有元表，返回nil，有元表则继续3 。
3. 判断元表有没有 `__index`，如果`__index`方法为nil，则返回nil；如果`__index`的值是一个table，则重复1、2、3；如果`__index`的值是一个function，则返回该函数的返回值。   


```
int create_account(lua_State *L){		double balance = luaL_checknumber(L,1);	Account **a = (Account**)lua_newuserdata(L,sizeof(Account));	*a = new Account(balance);	printf("construct! balance is: %lf\n",balance);	luaL_getmetatable(L,"Account");	lua_setmetatable(L,-2);//将userdata与Account metatable关联起来	return 1;}
```
#### 3、 C++ 类`Account`的方法映射到C函数

每一个函数原理都基本一致：   

+ 首先，从Lua栈顶获取`userdata`   
+ 取`userdata`保存的C++对象的指针   
+ 最后，对过C++对象指针访问对应方法   
```Account* checkaccount(lua_State *L, int narg){	luaL_checktype(L,narg, LUA_TUSERDATA);	void *ud = luaL_checkudata(L,narg,"Account");	luaL_argcheck(L,ud!= NULL,1,"user data error");	return *(Account**)ud;}int deposit(lua_State *L){	Account *a = checkaccount(L,1);//(Account**)lua_touserdata(L,1);	double amount = luaL_checknumber(L,-1);	(a)->deposit(amount);	return 0;}int withdraw(lua_State *L){	Account *a = checkaccount(L,1);//(Account**)lua_touserdata(L,1);	double amount = luaL_checknumber(L,-1);	(a)->withdraw(amount);	return 0;}int balance(lua_State *L){	Account *a = checkaccount(L,1);//(Account**)lua_touserdata(L,1);	double balance = (a)->balance();	printf("balance is: %lf\n",balance);	lua_pushnumber(L,balance);	return 0;}```

#### 4、注册

+ 创建一个`metatable` `Account`，`Account`将会关联到存储C++对象的`userdata`
+ 将第三步的C函数注册到`Account`表
+ 注册一个`Account`全局函数，可以理解为在Lua中的一个构造函数

```static const luaL_Reg methods[]= {	//{"Account",create_account},	{"deposit",deposit},	{"withdraw",withdraw},	{"balance",balance},	{NULL,NULL}};int luaopen_Account(lua_State *L){	luaL_newmetatable(L,"Account");	lua_pushvalue(L,-1);	lua_setfield(L,-2,"__index");//Account.__index = Account	luaL_setfuncs(L,methods,0);//register the methods to the table that is on the top	return 1;}static const luaL_Reg libs [] = {{"Account",luaopen_Account},{NULL,NULL}};void Register(lua_State* L){	const luaL_Reg *lib = libs;	for (;lib->func; lib++) 	{		luaL_requiref(L, lib->name, lib->func, 1);		lua_pop(L, 1);  /* remove lib */	}	lua_register(L,"Account",create_account);}   
```

#### 5、test.lua  


```
print("start");local a  = Account(100);print(a);print(a:balance());a:deposit(399);print("start");print(a:balance());print("end");
```    

## 三、C#类导入Lua  

C# 单个函数导入Lua比较简单与 C 导入Lua基本一致，此处就不赘述。C#类的导入则与C++差不多，但是怎么将C#对象存储于`userdata`是一个问题。有两种方案处理这个问题（仅本人所知）：

+ C# 对象可以通过`Marshal.StructureToPtr`转为`IntPtr`的指针，但是要求类添加`[StructLayout(LayoutKind.Sequential)]`标记，如此如果要导出一些非自定义类可能比较麻烦。   
+ 另一个方法，将一个`int`存储于`userdata`，并以`int`做key C#对象为value缓存于C#字典中，对于`userdata`的其它操作与C++一致。当lua调用该对象时，通过`Marshal.ReadInt32`函数，将返回的`userdata`转为一个`int`的key从C#对象字典中取出实际对象即可。     

接下来就方法二进行一下了解。

### 3.1 实例   
说明：

+ 将Lua虚拟机制作为Dll，并添加到Unity工程，本文直接使用的Slua来测试的
+ 本文环境为Unity

#### 1、将要导入Lua的C#类`LuaTestDemo`

```
    class LuaTestDemo:System.Object    {        public double Add(double a, double b)        {            Debug.Log("lua call the Add function!!");            double ret = a + b;            Debug.Log("==========: "+ret);            return ret;        }    }```    

#### 2、“构造”函数
与C++处理流程基本一致，**需要注意的是`userdata`的值是`100`，并将`LuaTestDemo`的实例做为`100`的value存入`Dictionary`**。

```      [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]       public static int CreateLuaTestDemo(IntPtr l)       {            Debug.Log("lua call the CreateLuaTestDemo function!!");            LuaDLL.luaS_newuserdata(l,100);//userdata is 100            LuaTestDemo o = new LuaTestDemo();            cache.Add(100,o);//map object to userdata            LuaDLL.puaL_getmetatable(l, "LuaTestDemo");//connect the userdata to LuaTestDemo metatable            LuaDLL.pua_setmetatable(l, -2);            return 1;        }```
#### 3、Wrapper.  
与C++一致，**需要注意的`userdata`获取的是一个`int`，并以`int`为key获取C#对象，然后访问对应方法;**     **通过[MonoPInvokeCallbackAttribute]标签让C语言可以直接调用C#函数**    

```		static LuaTestDemo checkuserdata(IntPtr l)        {            LuaDLL.puaL_checktype(l,1,LuaTypes.LUA_TUSERDATA);            IntPtr userdata = LuaDLL.puaL_checkudata(l, 1, "LuaTestDemo");            int index = Marshal.ReadInt32(userdata);            Debug.Log("userdata is: " + index);            LuaTestDemo o = cache[index];            return o;        }
                [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]        static public int AddWrapper(IntPtr l)        {            Debug.Log("lua call the AddWrapper function!!");            LuaTestDemo o = checkuserdata(l);            double a = LuaDLL.pua_tonumber(l, -1);            double b = LuaDLL.pua_tonumber(l, -2);            double ret = o.Add(a,b);            LuaDLL.pua_pushnumber(l, ret);            return 1;        }        ```
#### 4、注册
```             public static void Reg(IntPtr l)        {            LuaDLL.puaL_newmetatable(l, "LuaTestDemo");            LuaDLL.pua_pushvalue(l,-1);            LuaDLL.pua_setfield(l,-2,"__index");//LuaTestDemo.__index = LuaTestDemo            LuaDLL.pua_pushcclosure(l, AddWrapper, 0);            LuaDLL.pua_setfield(l,-2,"Add");            LuaDLL.pua_pop(l,1);                        LuaDLL.pua_pushcclosure(l, CreateLuaTestDemo , 0);            LuaDLL.pua_setglobal(l, "LuaTestDemo");        }    }
```    
#### 5、测试
```
IntPtr l = LuaDLL.puaL_newstate();		LuaTestDemoWrapper.Reg(l);		string lua = "" +		             "local d = LuaTestDemo();" +		             "local ret = d:Add(100,500);" +		             "local ret2 = d:Add(1000,5000);" +		             "print(\"ret is:\"..ret..\"   \"..ret2);" +		             "";		LuaDLL.pua_dostring(l,lua);
```

 

## 四、总结
+ 上文仅是一个入门的学习，没有涉及到内存的管理，内存的管理应该是比较核心的。
+ C/C++/C#导入Lua的本质可以简单理解为，宿主程序通过Lua栈从Lua获取参数，然后宿主程序将计算结果压入Lua栈供Lua读取