---
title: Blocks in iOS
date: 2017-03-19 11:01:20
categories: iOS
tags: [iOS, Dev]
---
> 最近由于工作原因，对iOS Block进行一次学习,本篇文章对Block的内存使用相关的内容简要整理一下，解释其中一些原理和使用Block需要注意的问题 。

<!--More-->

# 一、关于Block的内存表示
## 1.1、`clang -rewite-objc 源文件 -o 目标文件（可以为.txt）`
+ 首先我们来看一段常见源码，通过上述命令查看其中间文件：

```     

	typedef void (^ test1) (BOOL success, NSError *error) ;

	int main(int argc, char * argv[]) {
	    @autoreleasepool {
	        int base =10;
	        for(int i=0;i<10;i++){
	            int a =10;
	            // int *b  = (int*)malloc(10*sizeof(int));
	            void (^ test) (BOOL success, NSError *error)= ^(BOOL success, NSError *error) {
	                NSLog(@"sssss:%d,base:%d",success,base);
	            } ;
	            test(1,nil);
	           
	        }
	
	//        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
	    }
	}

```

+ 通过sublime打开目标文件会发现代码远不止上面那寥寥几行，不难从其中可以找到如下与Block相关的代码。

```

	struct __block_impl {
  		void *isa;
  		int Flags;
  		int Reserved;
  		void *FuncPtr;
	};
```

`struct __block_impl`就是block的真面目，结构还是相当清晰。`isa`对OC开发人员再熟悉不过，是表示其类型或者说本质的，block有三种类型`NSGlobalBlock`，`NSStackBlock`,`NSMallocBlock`，不同类型的block就是通过`isa`字段区分。然后就是`FuncPtr`，顾名思义就是指向某一函数的指针。其实block本质上与c语言中的函数指针是一样的。

```   

	struct __main_block_impl_0 {
	  struct __block_impl impl;
	  struct __main_block_desc_0* Desc;
	  int base;
	  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _base, int flags=0) : base(_base) {
	    impl.isa = &_NSConcreteStackBlock;
	    impl.Flags = flags;
	    impl.FuncPtr = fp;
	    Desc = desc;
	  }
	};
	
	static void __main_block_func_0(struct __main_block_impl_0 *__cself, BOOL success, NSError *error) {
	  int base = __cself->base; // bound by copy
	
	                NSLog((NSString *)&__NSConstantStringImpl__var_folders_rl_k02s7ny56tj7m7cg3kdrfpbh0000gn_T_main_0c0cda_mi_0,success,base);
	            }

	static struct __main_block_desc_0 {
	  size_t reserved;
	  size_t Block_size;
	} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
```
```

	int main(int argc, char * argv[]) {
	    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
	        int base =10;
	
	        for(int i=0;i<10;i++){
	            int a =10;
	
	            void (* test) (BOOL success, NSError *error)= (void (*)(BOOL, NSError *))((id (*)(id, SEL))(void *)objc_msgSend)((id)((void (*)(BOOL, NSError *))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, base)), sel_registerName("copy"));
	            ((void (*)(__block_impl *, BOOL, NSError *))((__block_impl *)test)->FuncPtr)((__block_impl *)test, 1, __null);
	
	    }
	}
```
以上这一段代码正是与源码相对应的`struct __main_block_impl_0`是此处block的真正实现，主要分为两部分一个是block的定义一个是block的一些描述。`__main_block_func_0`是具体执行的函数体。接下来就是要`main`函数体中初始化block，然后就大功告成可以便可享用block。
以上揭示了block的本质。
# 二、关于Block类型
## 2.1 Block的类型

|类|对象存储域|
|:--|:--|
|_NSConreteStackBlock|栈上|
|_NSConreteGlobalBlock|全局数据区（.data区）|
|_NSConreteMallocBlock|堆上|
>在ARC开启时，将只有 _NSConreteGlobalBlock与_NSConreteMallocBlock。因为ARC下编译器会自己管理对象，默认copy到堆上

其实Block的类型与其自动截获变量有关。

+ _NSConreteGlobalBlock类型
_NSConreteGlobalBlock类的Block，本质就与一个全局函数一样。不截获任何外部变量，只使用入参或者不使用任何参数，如：   

		void (^ blk )(void)= ^{print("Global Block\n")}

+ _NSConreteStackBlock类型   
一般来说Block除了Global Block外，首先他必须先是一个_NSConreteStackBlock，也就是配置在栈上的。 因为，Block变量首先得是一个变量，而代码中除了主动申请内存外其它变量都在栈上。

+ _NSConreteMallocBlock类型    

	**那么关于_NSConreteMallocBlock类型何时才用到呢？由于_NSConreteStackBlock是配置于栈上，所以其生命周期也随其变量在栈上的生命周期。但实际开发中往往需要Block的生命周期更长。所以Block提供了复制，_NSConreteStackBlock可以复制到堆上也就是_NSConreteMallocBlock，在堆上的变量生命周期与栈无关，不会随着栈上变量变化。block 通常不会在源码中直接出现，因为默认它是当一个 block 被 copy 的时候，才会将这个 block 复制到堆中。**
	
## 2.2 对Block进行copy的效果

|类|源存储域|复制效果|   
|:--|:--|:--|   
|_NSConreteStackBlock|栈上|从栈复制到堆|   
|_NSConreteGlobalBlock|全局数据区（.data区）|无效果|   
|_NSConreteMallocBlock|堆上|引用计数增加|
> 在ARC中一般不需要显示进行copy操作，编译器会自动判断。
  

# 三、__block变量

**Block中可以进行修改的值包括三种：**   
 
+ 全局变量   
+ 静态变量   
+ `__block`变量  

前两种不用说，因为其存储在全局区域访问自然无可厚非，本节主要说一下`__block`变量。还是原来的demo，仅将其中的变量`base`声明为`__block`

	 __block int base =10;

经过重写后，结果又大不相同

```

	struct __Block_byref_base_0 {
	  void *__isa;
	__Block_byref_base_0 *__forwarding;
	 int __flags;
	 int __size;
	 int base;
	};

	struct __main_block_impl_0 {
	  struct __block_impl impl;
	  struct __main_block_desc_0* Desc;
	  __Block_byref_base_0 *base; // by ref
	  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_base_0 *_base, int flags=0) : base(_base->__forwarding) {
	    impl.isa = &_NSConcreteStackBlock;
	    impl.Flags = flags;
	    impl.FuncPtr = fp;
	    Desc = desc;
	  }
	};
	static void __main_block_func_0(struct __main_block_impl_0 *__cself, BOOL success, NSError *error) {
	  __Block_byref_base_0 *base = __cself->base; // bound by ref
	
	                NSLog((NSString *)&__NSConstantStringImpl__var_folders_rl_k02s7ny56tj7m7cg3kdrfpbh0000gn_T_main_5fdc20_mi_0,success,(base->__forwarding->base));
	            }
	static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->base, (void*)src->base, 8/*BLOCK_FIELD_IS_BYREF*/);}
	
	static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->base, 8/*BLOCK_FIELD_IS_BYREF*/);}
	
	static struct __main_block_desc_0 {
	  size_t reserved;
	  size_t Block_size;
	  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
	  void (*dispose)(struct __main_block_impl_0*);
	} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
	int main(int argc, char * argv[]) {
	    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
	        __attribute__((__blocks__(byref))) __Block_byref_base_0 base = {(void*)0,(__Block_byref_base_0 *)&base, 0, sizeof(__Block_byref_base_0), 10};
	        for(int i=0;i<10;i++){
	            int a =10;
	
	            void (* test) (BOOL success, NSError *error)= ((void (*)(BOOL, NSError *))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_base_0 *)&base, 570425344)) ;
	            ((void (*)(__block_impl *, BOOL, NSError *))((__block_impl *)test)->FuncPtr)((__block_impl *)test, 1, __null);
	
	        }
	
	
	    }
	}
```    

不难发现一个`__block`变量竟然是一个结构体。注意`main`函数中的`__block`变量的初始化已经演变成如下代码：     

	__attribute__((__blocks__(byref))) __Block_byref_base_0 base = 
	{(void*)0,
	(__Block_byref_base_0 *)&base,
	 0, 
	sizeof(__Block_byref_base_0), 
	10
	};

### __block变量在Block内外都可以访问

注意结合该结构体声明分析。值得关注的是其`__forwarding`指针指向了自己本身。     
**<font color = red>__forwarding的存在是为了不管__block变量是在堆上还是栈上都可以访问该变量，因为在复制__block变量到堆上是会改变栈上__block结构中的__forwarding指向堆上的，而堆上的__block变量指向自己本身，因此堆上与栈上都能可以该变量。</font>**


# 总结
### 1、 Block是一个OC对象
### 2、__block变量可以有Block内外都可以访问，是由于`__forwarding`的存在
### 3、虽然本文是研究Block的内存表象，但对于Block的常见问题还是得提一下，注意Block循环引用问题