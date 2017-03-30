---
title: Python Dev
date: 2017-03-21 16:57:02
tags: [Python, Dev]
categories: Python
---
# Python 学习

<!--More-->

## 一、基础知识   

### 1.0、逻辑运算（补充）
 
|逻辑运算|python 表示|  
| -- | --|  
|与|and |  
|或|or|  
|非| not|  
|异或|^|

### 1.1、 关于list与tuple(元组),<font color = red> *数组*</font>

1、`list` 是可变，`tuple`是不可变，可以用`tuple`的地方尽量用`tuple`；  

2、`list`利用“[]”表示，`tuple`利用"()"表示  
3、数组python用numpy来做的：  

```
	import numpy
	matrix = numpy.zeros(shape=(5,2))#创建数组
	print matrix[i,j]#获取元素
```

### 1.2、关于dict,set

参考c++map,set

**CAUTION:** Python的集合(set)和其他语言类似, 是一个无序不重复元素集, 基本功能包括关系测试和消除重复元素. 集合对象还支持union(联合), intersection(交), difference(差)和sysmmetric difference(对称差集)等数学运算.由于集合是无序的,所以，sets 不支持 索引, 分片, 或其它类序列（sequence-like）的操作。Dict的key必须是不可变的

### 1.3、关于python的可变参数与关键字参数

1、可变参数就是传入的参数的个数是可变定义方式为`def func(*args)`。可变参数在函数调用时自动组装为一tuple。

2、关键字参数可以传入零个或多个含参数名的参数也就是键值对，定义方式为`def func(**kv)`。关键字参数在函数调用时组装成一个dict。

3、list,tuple可以在变量前面添加`*`来传入到参数为可变参数的函数，dict同样也可传入到参数为关键字参数的函数。

4、由于3，对于任意函数我们都可以通过`def func(*args,**kv)`来调用它，不管他是怎么样定义的。

5、python函数的参数顺序问题：必须函数，默认参数，可变参数，关键字参数  

### 1.4、Python所谓的高级我切片与matlab的取第几行几列类似


### 1.5、迭代使用`for ... in `或者`for value in d.iteatervalues()`同时也可以用`for k,v in d.iteritems()`

```

	d  = { a :1, b :2}
	
	for key in d:
	
	    print key
```  

1、但是要使用迭代一个对象，那么这个对象要是**可迭代的**，<font color = red>方法：可以通过collections 模块的iterable进行判断</font>  


### 1.6、生成器(Generator)

由于通过列表生成器生成一个列表时，受到内存的限制，列表的容量是有限的。所以如果可以按照一种算法进行推算出来，那就不必创建完整的list从而可以节省大量空间。创建generator的方法有多种：  

第一种： 


```

	l = [ x*x for x in range(1,11) ]
	
	g = (x*x for x in range(1,11))   # g就是一个generator，与列表生成器不同的是[ ]与（ ）的问题  
	
	g.next()  # generator保存的是算法每次调用next()实际是在计算下一个元素，直到计算到最后的元素，当没有元素时就会抛出StopIteration的错误
	
	for n in g: # 由于不断调用next（）函数并不是很方便，由于generator是可迭代对象，因此正确的使用方法是使用迭代
	
	    print n
```  

第二种：

在函数定义中如果包含yield关键字，那么这个函数就是一个generator

```

	def fib(max):
	
	    n,a,b = 0,0,1
	
	
	    while n<max:
	
	
	        yield b      # yield关键字
	
	
	        a,b =b ,a+b
	
	
	        n = n+1
```  




### 1.7、python高函数

所谓的高阶函数就是一个函数接收函数作为参数

### 1.8、map/reduce/filter

[关于MapReduce文章]( http://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf)    

+ map/reduce demo  




```

	def str2int(s):
	
	    def fn(x,y):
	
	        return x*10+y
	
	
	    def char2num (s) :
	
	
	        return {  0  : 0 ,  1  : 1 ,  2  : 2 ,  3  : 3 ,  4  : 4 ,  5  : 5 ,  6  : 6 ,  7  : 7 ,  8  : 8 ,  9  : 9 }[s]
	
	    return reduce(fn,map(char2num,s))

```  

<font color=red>**注意map,reduce都接收一个函数与一个list，因此第5行代码就好理解了**   </font>    

+ filter 与map/reduce一样都 是高阶函数都是接收一个函数与list变量,filter是根据传入函数作用于每一个元素返回true/false来决定这个元素是否保留

```

	def is_odd(n):
	
	    return n%2 ==1
	
	
	filter(is_odd,[1,2,3,4,5])

```   




### 1.9、**funciton.partial偏函数**，把一个函数的某些参数给固定住，返回一个新函数，那么调用 这个函数就会简单一些。当一个函数的参数个数太多需要简化的时候可以使用functools.partial创建一个偏函数。

```

	import functools
	
	int2 = functools.partial(int,base = 2) #int2就是一个新函数只有一个参数
```      

### 1.9.1 `@`语法


开门见山，直上代码

```

	def log(func):
	
	        def wrapper(*args,**kw):
	
	
	            print  call %s %func.__name__
	
	
	            return func(*args,**kw)
	
	
	return wrapper

```  

`log`函数接收一个函数作为参数，然后返回一个函数，这就叫做decorator。那么怎么用这个decorator呢？代码直上：  

```

	@log
	
	def now():
	
	    print  sdfdsfds 


```  

`@`语法相当于：`now = log(now)`     
  
### 1.10、python用模块（module）来组织工程

1、使用模块可以避免函数与变量名的冲突。一个.py文件 就是一个模块。

2、为了避免模块名的冲突python引入package，本质就是按目录来组织模块

3、package目录下一定要有一__init__.py模块，否则认为普通目录  

4、注释也可采用  

```

		‘’‘
	
		注释
		
		’‘’

```

### 1.11、关于`if __name__== main:`

当我们用命令行运行模块的时候，Python解释器就会将一个特殊变量`__name__`置为`__main__`，而在其它地方导入该模块这个判断就会失败，也就是说这条语句主要是便于我们使用命令行。  

### 1.12、作用域

private访问控制在python里通过`_`或者`__`前缀来实现。这里所说的private只是一种编程习惯，因为python并没有private的控制访问。

### 1.13、python面向对象编程(Object Oriented Programming,OOP)

话不多说直接上demo

```

	class Student(object):                                 #表示Student从object继承 
	
	        def __init__(self,name,score):            #__init__类于构造函数，且第一个参数永远是self，表示创建实例本身。而且实例化对象时一定要传入与__init__参数对应的参数
	
	
	                self.name = name
	
	                self.score = score
	
	                super(Student,self).__init__()    #表示调用父类方法
	
	
	        
	
	
	        def print_score(self):
	
	
	                print  %s,%s %(self.name,self.score)
	

```    

 1、`__init__`类于构造函数，且第一个参数永远是self，表示创建实例本身。而且实例化对象时一定要传入与__init__参数对应的参数。


2、成员函数与普通函数不一样的地方就是第一个参数永远是self

3、private访问控制权限就是在变量前面添加`__`前缀。`_`在类中视为private但是其实是可以访问的，只是按约定视为private.

4、private变量其实也是可以访问的，可以通过`_类名_private变量`如`_Student_name`  

5、  super(className,self).function()      父类方法调用<font color = red> 如果是多继承且多个父类有同样的方法则按顺序调用</font>

### 1.14、关于python的多态

由于python里不需要事先声明变量的类型，因此多态实现起来就简单得多：  

```

	def run(animal):
	
	    animal.run()


```  

代码中只要传入的参数有run这个方法就可以，<font color = red>**不管是不是animal还是子类或者不是子类**</font>，这样子就与c++里的多态一样。其实本质上是由于python本来就是动态语言。  

### 1.15、获取对象信息

1、type可以判断基本类型的所属类型

```

	import types 
	
	type( ssdfdf )

```

2、isinstance可以判断基本数据类型还可以判断class类

3、dir可以获取对象的所有属性和方法

4、getattr(),setattr(),hasattr()可以操作一个对象的属性，顾名思义  

### 1.16、python动态绑定方法

```

	def set_age(self,age):
	
	        self.age = age
	
	
	from types import MethodType
	
	s.set_age = MethodType(set_age,s,Student)          #给实例新增方法
	
	
	
	
	Student.set_age = MethodType(set_age,None,Student)    #给类新增方法

```   

1、<font color = red>**`__slot__`**</font>关键字

如果想对class限制只允许给类的实例添加属性时，我们可以使用`__slot__`，但是不遗传

```

	class Student(object):
	
	        __slot__ = ( name , age )  #用tuple定义允许修改的属性名称


```

###1.17、`@property`关键字

`@property`负责将一个方法变成属性调用，而与一个属性不同的是属性并没有检查是否合法，而方法是可以进行检查的。

```

	class Student(object):
	
	        @property
	
	
	        def score(self):
	
	
	                return self._score
	
	
	        
	
	
	        @score.setter
	
	
	        def score(self,value):
	
	
	                if():
	
	
	                        .....#参数检查
	
	
	                self._score = value
	
	
	
	
	
	>>>s = Student()
	
	>>>s.score = 10 #实际上是调用的s.set_score()
	
	>>>s.score #实际上调用 的是s.get_score()

```

`@property`把一个方法变成属性，`@xxx.setter`把一个方法变成属性赋值，与OC的property有一点相似

### 1.18、多重继承

### 1.19、定制类

### 1.20、元类

1、type动态创建一个对象

```

	hello = type( Hello ,(object,),dict(hello = fn))

```  

type接收三个参数：

+ class名字

+ 参数2 为父类，是一个tuple（object,）

+ dict(hello = fn),绑定函数到类的方法

   

2、metaclass元类，（OC中也有这个概念）   

### 1.21、异常捕获`try:....except...finally...`

```

	try:
	
	    print  try.... 
	
	
	    r = 10/0
	
	
	    print  reuslt: ,r
	
	
	except ZeroDivisionError ,e:        #还可以多级捕获
	
	    print  error: ,e
	
	
	finally:
	
	    print  finally... 
```   

1、可以通过`raise`抛出异常，`raise`后面不带参数则将错误原样抛出，同样`raise`也可以修改抛出错误的类型  

### 1.22、调试

1、print打印信息

2、asser断言

3、logging推荐，可以定义log级别：debug,info,waning,error ,可以指定输出文件`logging.config.fileConfig(filename)`,[filename规则](https://docs.python.org/2/library/logging.config.html#logging-config-fileformat)  ，给一个log.conf

```
	[loggers]
	keys=root
	
	[handlers]
	keys=consoleHandler,fileHandler
	
	[formatters]
	keys=simpleFormatter
	
	[logger_root]
	level=DEBUG
	handlers=consoleHandler,fileHandler
	
	[handler_consoleHandler]
	class=StreamHandler
	level=DEBUG
	formatter=simpleFormatter
	args=(sys.stdout,)
	
	[handler_fileHandler]
	class=logging.handlers.TimedRotatingFileHandler
	level=DEBUG
	formatter=simpleFormatter
	args=('./log/sample.log','D',1,14)
	#args=('/data/iTOP_ROOT/dev/log/admin/pack/sample.log','D',1,14)
	
	[formatter_simpleFormatter]
	format=[%(asctime)s][pid %(process)d][%(levelname)s][%(filename)s:%(lineno)d]%(message)s
	datefmt=
```

使用方式：  

```
	logging.config.fileConfig('./log.conf')
	logger = logging.getLogger('root')
```

4、[pdb]( http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/00138683229901532c40b749184441dbd428d2e0f8aa50e000)   

### 1.23、单元测试  

### 1.24、文档测试（测试注释了的代码--python交互式代码）

### 1.25、IO编程

1、文件读写   

```python

    f = open(filename,r)


    f.read()


    f.close


```    

为了保证会调用`close()`同时也是为了简单化，Python引入了`with`语法来帮助我们调用`close()`


```python

	with open( filename ,r ) as f:

   		 print f.read()


```   

2、[操作文件和目录]( http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013868321590543ff305fb9f9949f08d760883cc243812000)    

3、序列化   

+ python中序列化提供了两个模块来实现：`cPickle`和`pickle`(有`c`开头的就是用c语言实现的)[pickle参考]( http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/00138683221577998e407bb309542d9b6a68d9276bc3dbe000)   

+ json，`pickle`的问题在于序列化只能用在python中不能进行传递或者其它语言来解释，json刚好可以解决这个问题。

| json | python |  
| ---- | ---- |     
| { } | dict |  
| [ ] | list |  
| "string" |  str 或者u unicode  |
| 123.4 | int 或者 float |  
| true/false | Ture /False |
| null | None |   

```python

	import josn
	
	d = dict(name = 'Bob',age = 10,score = 88)
	
	json.dumps(d)  #dump意为转储
	
	json.loads(str)#转成dict python对象

```  

<font color = red>**CAUTION:**</font>json反序列化得到字符串对象都是unicode不是str。  

+ class 序列化（json的进阶）   

```python
	json.dumps(s, 
	default
	=lambda obj: obj.__dict__)#当class有__slot__属性时不能用 

```      





### 1.26、[进程与线程]( http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013868323401155ceb3db1e2044f80b974b469eb06cb43000)     

1、进程

+ Unix/Linux下可以调用`fork()`实现多进程

+ 跨平台的多进程，可以使用multiprocessing  

+ 进程间的通信可以使用Queue,Pipes  

2、线程，python中有两个模块（thread,threading，后者是高级，我们一般使用高级）

+ 线程与进程最大的不同是，同一个变量在进程中是各自有一份copy互不影响，而多线程中所有的线程共享进程的变量，是相互影响的。因此在多线程中我们使用Lock来保证变量的不相互影响   




```python

	import time, threading 
	
	balance = 0
	lock = threading.Lock()
	def change_it(n):    
	
	        global balance   
	
	         balance = balance + n   
	
	         balance = balance - n 
	
	 def run_thread(n):    
	
	        for i in range(100000):
	               lock.acquire()     #先获取锁
	               try:                #try...finally保证不管什么情况下都会释放locK
	                   change_it(n)
	               finally:
	                   lock.release()  #释放锁
	if __name__ ==  __main__ :
	      t1 = threading.Thread(target=run_thread, args=(5,))  
	
	      t2 = threading.Thread(target=run_thread, args=(8,))   
	
	      t1.start()   
	
	     t2.start()   
	
	     t1.join()    
	
	    t2.join()    
	
	    print balance
```    

+ 在多线程下每一个线程要保持自己的数据时，我们可使用ThreadLocal, **ThreadLocal最常用的地方就是为每一个线程绑定一个数据库链接，http请求，用户身份信息等**




```python

	import threading
	
	local_shcool = threading.local()  #每一个线程都可以写，且相互不影响，可以理解为一个以thread为key的dict
	
	def pro_std():
	
	    print hello,%s (in %s) %(loca_school.student,threading.current_thread().name)
	
	
	def pro_thread(name):
	
	    local_shcool.student = name
	
	
	    pro_std()
	
	t1 = threading.Thread(target = pro_thread,args = ( Alice ),name =  a )
	
	t2 = threading.Thread(target = pro_thread,args = ( Bob ),name =  B )
	
	t1.start()
	
	t2.start()
	
	t1.join()
	
	t2.join()
	
```  

### 1.27、分布式编程   

[参考]( http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832973658c780d8bfa4c6406f83b2b3097aed5df6000)   

### 1.28、正则表达式

### 1.29常用内嵌模块  

1、collections

2、base64:base64是一种任意二进制到文本字符串的编码方法。**关于base64的理解：将3个字符串转成4个字符串，3 \* 8 = 4\*6，因此叫base64**  

3、hashlib摘要算法库，md5,SHA1。md5结果是128bit,SHA1结果是160bit

```python
	
	import hashlib
	
	md5 = hashlib.md5()
	
	md5.update(  hello world )
	
	md5.update( hdhhd )
	
	print md5.hexdigest()

```    

SHA1与md5的调用 一样将上述代码改成SHA1即可  
## 2、python进阶  
### 2.1、Range与XRange的区别
在 Range的方法中，它会生成一个**list**的对象，但是在XRange中，它生成的却是一个**xrange的对象**，当返回的东西不是很大的时候，或者在一个 循环里，基本上都是从头查到底的情况下，这两个方法的效率差不多。但是，当返回的东西很大，或者循环中常常会被Break出来的话，还是建议使用 XRange，这样既省空间，又会提高效率。

### 2.2、ArgumentParser处理数组传入

```
	ap = argparse.ArgumentParser()
    ap.add_argument('-p','--plugins',required = True, nargs = '+', help = 'the channel which you will test')
```


## 3、python实战
### 3.1、[python制作gif]( http://python.jobbole.com/81185/)，然后可以在ppt导入    
### 3.2、Tkinter UI  
1、动态更新UI，使用`config`   
```python
	root = Tk()
	btn = Button(root,text ='test')
	btn.pack()
	btn.config(state = 'disabled'）
```   
关于布局有三种，pack,grid,place[参考]( http://www.tutorialspoint.com/python/tk_pack.htm)   


## 4、opencv python   
   
<span id = "4"> </span>
### 4.1、反转黑白图片
```
	thresh = cv2.thrsehold(image,122,255,cv2.THRESH_BINARY_INV)[1]
```   

### 4.2、[python进行条形码检测]( http://python.jobbole.com/81130/)    
### 4.3、cv2  crop图片  
```python
	img = image[y:y+h,x:x+w]
```   
### 4.4、由文本生成图片，[参考](http://python.jobbole.com/81983/)     
### 4.5、图片生成汉字库，关于怎么样将汉字切片可以[参考](http://python.jobbole.com/81985/)  