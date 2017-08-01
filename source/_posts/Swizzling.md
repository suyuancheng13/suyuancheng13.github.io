layout: method
title: Swizzling
date: 2017-07-25 15:06:57
tags: [iOS, Dev]
categories: iOS
---

> iOS Method　Swizzling学习笔记    

<!--More-->

# 一、前言
上周，由于工作原因接触到了Google的Firebase插件，细读其文档发现竟然使用了iOS界的黑科技——Swizzling。受其毒害受其鼓舞，受其毒害是指它暴露了我们项目的一个缺陷需要规避；受其鼓舞是指它竟然使用了Swizzling。加上很早就与老司机聊到过我们项目使用Swizzling必要性与可能性，所以近日决定试一试法。    
我们项目的基本框架是插件化，其有一个问题是部分插件需要在`didFinishLaunch`中做初始化，如果不按第三方插件指引可能会出现意想不到的结果（原因并不太清楚）。但是我们的目前的框架并不能完成这一需求，我们的插件并没有APP生命周期的概念，只是一个单纯的类而已。`didFinishLaunch`是App启动时调用的第一个方法此时我们的插件都还没初始化，根本没法做到第三方的初始化。如果，采用Swizzling对AppDelegate的方法进行Hook，那么就能很好地完成这一需求了。另外，还可以省去用户对我们的项目的AppDelegate的关注（目前需要用户在AppDelegate中显示调用）。     
说得好像真的一样，但是目前我们还需要关注几个问题：   

+ 问题一、多个插件同时进行swizzling结果会怎么样   
+ 问题二、该以什么样的形式供项目使用
+ 问题三、有没有更好的方案

# 二、Method Swizzling原理
## 2.1、Runtime   
+ 首先，看一下OC中Class的结构      

		struct objc_class {
	    Class isa  OBJC_ISA_AVAILABILITY;
	
		#if !__OBJC2__
	    Class super_class                                        OBJC2_UNAVAILABLE;
	    const char *name                                         OBJC2_UNAVAILABLE;
	    long version                                             OBJC2_UNAVAILABLE;
	    long info                                                OBJC2_UNAVAILABLE;
	    long instance_size                                       OBJC2_UNAVAILABLE;
	    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
	    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
	    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
	    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
		#endif
	
		} OBJC2_UNAVAILABLE;   

+ 其中，`methodLists`就是方法列表
		
		struct objc_method_list {
	    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;
	
	    int method_count                                         OBJC2_UNAVAILABLE;
		#ifdef __LP64__
		    int space                                                OBJC2_UNAVAILABLE;
		#endif
		    /* variable length structure */
		    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
		} 

+ `method`的结构如下：     
		
		struct objc_method {
	    SEL method_name                                          OBJC2_UNAVAILABLE;
	    char *method_types                                       OBJC2_UNAVAILABLE;
	    IMP method_imp                                           OBJC2_UNAVAILABLE;
		}  
+ `IMP`定义：

		typedef id (*IMP)(id, SEL, ...);    

源码中可以很容易知道在Objective C中的一个方法，其实最终会转换成`objc_method`的结构体，而结构体中清晰可以看到`SEL`与`IMP`。简而言之，就是一个OC的类有一个`SEL`表示函数名，`IMP`（函数指针）指向函数的具体实现。调用一个函数时便是通过`SEL`找到对应的`IMP`。所谓的Swizzling就是在类的消息分发列表也就是上述methodlist中交换两个方法的`IMP`。       
[附swizzling图片]



# 三、Method Swizzling实现    

```   

	 +(void)load {
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        Class originalClass = NSClassFromString(@"AppDelegate");
	        Class swizzledClass = [self class];
	        
	        SEL originalSelector = NSSelectorFromString(@"application:didFinishLaunchingWithOptions:");
	        SEL swizzledSelector = @selector(firebase_application:didFinishLaunchingWithOptions:);
	        
	        Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
	        Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
	        
	//        
	//                BOOL registerMethod = class_addMethod(originalClass,
	//                                                      swizzledSelector,
	//                                                      method_getImplementation(swizzledMethod),
	//                                                      method_getTypeEncoding(swizzledMethod));
	//                if (!registerMethod) {
	//                    return;
	//                }
	//        
	//                swizzledMethod = class_getInstanceMethod(originalClass, swizzledSelector);
	//                if (!swizzledMethod) {
	//                    return;
	//                }
	        
	        BOOL didAddMethod = class_addMethod(originalClass,
	                                            originalSelector,
	                                            method_getImplementation(swizzledMethod),
	                                            method_getTypeEncoding(swizzledMethod));
	        if (didAddMethod) {
	            class_replaceMethod(originalClass,
	                                swizzledSelector,
	                                method_getImplementation(originalMethod),
	                                method_getTypeEncoding(originalMethod));
	        } else {
	            method_exchangeImplementations(originalMethod, swizzledMethod);
	        }
	    });
	}     
	
	- (BOOL)firebase_application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	    ///注意此时的self已经是AppDelegate了
	    [FIRApp configure];
	    [FIRMessaging messaging].delegate = self;
	    NSLog(@"in : %s", __func__);
	    return [self firebase_application:application didFinishLaunchingWithOptions:launchOptions];
	}
```    

其中，需要注意的是：   

+ 第一，`originalClass`有实现`originalSelector`方法。这种情况下`class_addMethod`会返回false，这种情况也是最简单的将两个方法利用`method_exchangeImplementations`进行交即可。
+ 第二，`originalClass`没有实现`originalSelector`方法。`class_addMethod`会返回true。这里需要注意的是**在代码最开始是获取到了`originalSelector`的，原因是获取的是父类的实现，这里的没有实现其实是指子类没有`override`，所以这里的`class_addMethod`效果是为子类重写了父类的方法** 。然后就是利用 `class_replaceMethod`将源方法的实现替换成交换的方法实现。
+ 第三，`firebase_application: didFinishLaunchingWithOptions:`中代码调用`[self firebase_application:application didFinishLaunchingWithOptions:launchOptions];`这里需要注意的是这里不会形成死循环（因为其实它的实现已经被交换了），而且是必须进行此句调用的，因为我们不能修改系统的原始状态。
> 注释的部分是自定类的处理部分，因为类的Category会自动将`swizzled`方法添加至类中而自定义则需要手动添加     

## 3.2、Category VS 自定类

对一个类的方法进行Swizzling一般有两种方法，一种实现这个类的`Category`；另外一种就是自定一个类。但是这两种方式还是有一定的区别的。  

+ Category    
Category方法定义的方法都自然而然地添加到原来类了，特别是需要实现某些第三Delegate时比较方便（Firebase就需要实现它的Delegate）。    
+ 自定义类    
其实Swizzling本质是将两个方法的实现进行交换，它的context应该是相同。所以一般会将自定类的实现先动态添加至需要被交换的类中。

**个人认为Category方式会更加合适。**

## 3.3、注意点

+ Swizzling最好在`+load`中做， 关于`+load`与`+initialize`的区别可以参考[Objective-C +load vs +initialize](http://blog.leichunfeng.com/blog/2015/05/02/objective-c-plus-load-vs-plus-initialize/)
+ 使用`dispatch_once`处理Swizzling部分代码，以保证代码只会执行一次且线程安全。  
+ Swizzle后记得在交换的方法里调用被交换的方法。
# 四、Swizzling 与 load+Notification
## 4.1、 load+Notification
这里所言的load+Notification是参考xx项目解决方案。简而言之，在`load`方法中监听AppDelegate的方法，然后在Listener收到通知时进行相关处理。但是个人认为load+Notification有以下几点不足之处：
   
+ load+Notification**监听方法需要是类方法**，另外需要注意的是**在OC中类方法中`self`为`Class`**与`Object`不一样。因为`load`是类方法则监听方法必须为类方法。
+ 基于上一点，如果需要实现第三Delegate时，是无法满足需求的。因为一般Delegate方法都不会是类方法    

优点：    

+ load+Notification是系统API的常规调用不存在任何风险

## 4.2、Swizzling
Swizzling刚好能解决load+Notification的一些缺陷，同时由于过于灵活且涉及一些非常规操作，出现问题的可能性自然就会增加。

# 五、示例
## 5.1、多个插件进行Swizzling  
+ 示意图
+ 运行结果
# 六、展望
Swizzling称为iOS的黑魔法，利用得当会给项目带很多好处，但是如果不当也可能会造成灾难性问题。因此，如果要用好此方法不仅仅是会写上面的代码还需要在联调迅速解决问题，这就需要对其本质做更多的探索，只有对它了如指掌才能发挥其最大潜力。