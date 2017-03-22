---
title: iOS Dev
date: 2017-03-18 11:08:00
tags: [iOS, Dev]
categories: iOS
---

> iOS 学习笔记

<!--More-->
## 一、iOS基础知识
### 1、语法

1、类方法里是不可以调用普通方法的。

2、`extension`与`catogry`的区别就是括号中有没有名字。另外`catogry`只可以增加方法不能增加属性，`extension`是可以增加属性的   

3、`self`的赋值一定要在`initxxx`系列函数里调用，不能在别的函数里调用，`initialxxx`都不行


### 2、关于在一个工程中添加自定义framework

1、添加方式：File->New->Target->Cocoa Touch Framework
2、真机测试时可会出现dyld:Library no loaded:....   
**解决方法**：在app对应的Target->General->Embedded Binaries 添加自定义framwork

### 3、Framework 命令行编译

利用xcode tools[参考](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html),命令如下：

> xcodebuild clean build -project /Users/yuanchengsu/Desktop/iMSDK/iMSDK/iOS/IMSDK/IMSDK.xcodeproj 
-target IMSDKCoreKit -configuration Release  -xcconfig /Users/yuanchengsu/Desktop/iMSDK/iMSDK/iOS/IMSDK/Demo/Configurations/IMSDKGNU++98.xcconfig -sdk iphoneos9.1


### 4.[KVO]( https://www.mikeash.com/pyblog/friday-qa-2009-01-23.html)     
### 5.[ios https](http://io.diveinedu.com/2016/01/09/iOS%E5%BA%94%E7%94%A8%E7%BD%91%E7%BB%9C%E5%AE%89%E5%85%A8%E4%B9%8BHTTPS.html)
### 6.runloop     
runloop只有在开启第二个线程的时候才考虑使用runloop，而且也并非所有的线程需要runloop
+ 使用runloop的情况    
  + use ports or custom input sources to communicate with other threads
  + use timers on the thread
  + use any of the `performSelector...`
  + keep the thread around of perform periodic tasks     
  
+ performselector在后台执行时，必须开启一个runloop，否则调用不了     
+ runLoop有两套API，NSRunloop,CFRunLoopRef     
+ runLoop在获取时，就是创建
+ 一个runloop有两个CFRunloopSource,source0:处理UIEvent，CFSocket,source1:mach port,CFMachPort,CFMessagePort;一种mode:default,tracking,common     

### 7.copy与retain    
+ copy是指拷贝内容    
+ retain是指拷贝指针     
+ copy需要对象遵守NSCopying协议的    
+     

### 8.关于xcode查看汇编的方法   
+ xcode 7.x: Debug ==> Debug Workflow ==> Show Disassembly when Debugging     
+ xocde 7.x之前： Product ==> Debug Workflow ==> Show Disassembly when Debugging    

### 9.oc严格单例

创建对象的步骤分为申请内存(alloc)、初始化(init)这两个步骤，我们要确保对象的唯一性，因此在第一步这个阶段我们就要拦截它。当我们调用alloc方法时，oc内部会调用allocWithZone这个方法来申请内存，我们覆写这个方法，然后在这个方法中调用shareInstance方法返回单例对象，这样就可以达到我们的目的。拷贝对象也是同样的原理，覆写copyWithZone方法，然后在这个方法中调用shareInstance方法返回单例对象。看代码吧：

```

	#import "Singleton.h"  
	  
	@implementation Singleton  
	  
	static Singleton* _instance = nil;  
	  
	+(instancetype) shareInstance  
	{  
	    static dispatch_once_t onceToken ;  
	    dispatch_once(&onceToken, ^{  
	        _instance = [[super allocWithZone:NULL] init] ;  
	    }) ;  
	      
	    return _instance ;  
	}  
	  
	+(id) allocWithZone:(struct _NSZone *)zone  
	{  
	    return [Singleton shareInstance] ;  
	}  
	  
	-(id) copyWithZone:(struct _NSZone *)zone  
	{  
	    return [Singleton shareInstance] ;  
	}  
	  
	@end  
```

## 二、关于certificates 和provisioning proflies

1、certificates：key(private key)

2、provisioning profiles:(public key)

 + appid(bundle identifier) 


 + devices(uuid)


 + certificates(private key)  

3、.p12是证书导出保存的形式     
4、wwdr证书失效会导致自己证书无效


## 三、关于Clang 与LLVM  


Clang 是一个 C++ 编写、基于 LLVM、发布于 LLVM BSD 许可证下的/C++/Objective C/Objective C++ 编译器。LLVM 是 Low Level Virtual Machine。具体介绍参考[clang and llvm]( http://objccn.io/issue-6-2/)

+ 命令clang -E 是用于宏定义展开  

   

## 四、反馈系统UI设计问题：





1、toolbar的设计问题，当用storyboard进行布局的时候要做到适配效果子在UIbarbuttonItem之间加入flexispace才能做到适配。

2、关于storyboard initialViewcontrollerwithIdentiy的问题，当view还没出现的时生成的对象的成员都会是空的，只有到显示的时候才会出现。解决要对其进行参数传递的时候要采用property方法或者利用dispatch_after方式来做

3、关于ipad版本浮动窗口形式的问题。当在ipad版本下设置Modalpresentationstyle就可以达到这种效果
4、关于storyboard的问题。一个app中可以有多个storyboard，通过instantiateviewcontrollerwithidentiter方式 来进行实例化。要注意的是storyboard要放到 mainbundle目录下
5、消除返回按钮上的文字，利用self.naviagtionitem.backbuttonitem = [[uibarbuttom alloc ]initwithtitle:@""] style:

target:action];[参见]( http://www.cnblogs.com/ygm900/p/3659619.html )  

## 五、关于isKindOfClass与isMemberOfClass的区别：

1、isKindOfClass，是判断一个对象是属于哪个类~~ 型，一直追溯到父类~~ 。或者子类的实例

2、isMemberClass，是判断一个对象属于哪个类~~ 型，不追溯到父类!~~ 。   
## 六、Core Data编程
1、core data 与sql的一些对应关系：

| 功能 | sql |core data| 
| -- | -- |--|  
|表头|表结构,实体，entity|NSEntityDescription|   
|表中的一行数据|记录|NSManagedObject| 
|查询|查询,select ...|NSFetchRequest|   
|持久化|表存储,。。。|NSPersistentStoreCoordinator|   
|数据库的设计，也就是.xcodemodel文件|数据库模型,。。。|NSManagedObjectModel|   
|操作上下文|数据库操作|NSManagedObjectContext|  
[更详细的底层操作](http://objccn.io/issue-4-1/)       
2、`NSManagedObjectContext`
`NSManagedObjectContext`是程序员主要接触的一个类，也就是所有的操作基本上都通过它来操作的，底层是怎么实现的我们不需要去关心  
3、`NSPersistentStoreCoordinator`
`NSPersistentStoreCoordinator`才是最终的操作实现都是才是核心,[参考](http://objccn.io/issue-4-1)   
4、[编程参考](http://iiiyu.com/2013/03/29/learning-ios-notes-eighteen/)
## 七、关于异步加载问题
1、`nsdata datawithcontentsofurl`是一个同步加载方法，因此要使用此方法要采用异步加载方法。可以适用于加载图片等；
2、注意`AFNertworking success/failure block` invoked in main thread。如果 没有进行队列设置默认会返回到主线程去。 
 


	if(sucess)
	{
	dispatch_group_async(self.completionGroup?:http_request_operation_completion_group(),self.completionQueue?:dispatch_get_main_queue(),^{});
	}   
     


## 八、关于 strong,weak引用修饰符  

1、strong相当于手动引用计数（manual reference count）中的retain，拥有对象直到对象释放。
2、weak与strong刚好相反，weak并不持有对象，而且当对象释放时weak我修饰的对象会自动赋值为nil。例如：   



	id _weak obj = [[NSObject alloc]init];//错误，weak不能持有对象	
	id _weak oo = nil;
	{
	   id strong object = [[NSObject alloc]init];
	   oo= object
	   NSLog(@"%@",oo)//此处对象是存在的
	}
     


## 九、[关于draweRect消耗内存](http://bihongbo.com/2016/01/03/memoryGhostdrawRect/)  
要想搞明白这个问题，我们需要撸一撸在`iOS`程序上图形显示的原理。在`iOS`系统中所有显示的视图都是从基类`UIView`继承而来的，同时`UIView`负责接收用户交互。**但是实际上你所看到的视图内容，包括图形等，都是由`UIView`的一个实例图层属性来绘制和渲染的，那就是`CALayer`。**
**最终我们的图形渲染落点落在`contents`身上**![如图](http://7xkdhe.com1.z0.glb.clouddn.com/drawRect3.001.png)。
`contents`也被称为寄宿图，除了给它赋值`CGImage`之外，我们也可以直接对它进行绘制，绘制的方法正是这次问题的关键，通过继承`UIView`并实现`-drawRect:`方法即可自定义绘制。`-drawRect:` 方法没有默认的实现，因为对`UIView`来说，寄宿图并不是必须的，`UIView`不关心绘制的内容。如果`UIView`检测到`-drawRect:`方法被调用了，它就会为视图分配一个寄宿图，这个寄宿图的像素尺寸等于视图大小乘以`contentsScale`(这个属性与屏幕分辨率有关，我们的画板程序在不同模拟器下呈现的内存用量不同也是因为它)的值。这也就是`-drawRect:`消耗内存的原因。

**解决方案：**使用CAShapeLayer,另外与屏幕大小的画板可以算出消耗内存几M左右，可以接受   
## 十、[关于设置圆角问题](http://www.cocoachina.com/ios/20160301/15486.html)  


由于设置圆角发生离屏渲染，所以对    

## 十一、[数据持久化]( http://casatwy.com/iosying-yong-jia-gou-tan-ben-di-chi-jiu-hua-fang-an-ji-dong-tai-bu-shu.html)

1.使用 archive的对象需要实现<NSCoding>里的方法，关于NSCoder怎么使用参考如下 ：   



	- (	void)encodeWithCoder:(NSCoder *	)aCoder {
	    [aCoder encodeObject:self.firstName forKey:PERSON_KEY_FIRSTNAME];
	    [aCoder encodeObject:self.lastName forKey:PERSON_KEY_LASTNAME];
	    [aCoder encodeFloat:self.height forKey:PERSON_KEY_HEIGHT];
	}
	- (
	id
	)initWithCoder:(NSCoder *
	)aDecoder {
	    self 
	=
	 [super init];	    
	if
	 (self !=
	 nil) {
	        self.firstName 
	=
	 [aDecoder decodeObjectForKey:PERSON_KEY_FIRSTNAME];
	        self.lastName 
	=
	 [aDecoder decodeObjectForKey:PERSON_KEY_LASTNAME];
	        self.height 
	=
	 [aDecoder decodeFloatForKey:PERSON_KEY_HEIGHT];
	    }	    
	return
	 self;
	}
   

## 十二、离屏渲染  



设置了以下属性时，都会触发离屏绘制：    

+ shouldRasterize（光栅化）  
+ masks（遮罩）    
+ shadows（阴影）   
+ edge antialiasing（抗锯齿）  


+ group opacity（不透明）   


+ 还有一种特殊的离屏渲染，`cpu`渲染，当我们使用drawRect时会触发







