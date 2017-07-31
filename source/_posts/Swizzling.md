layout: method
title: Swizzling
date: 2017-07-25 15:06:57
tags: [iOS, Dev]
categories: iOS
---

> iOS Method　Swizzling学习笔记    

<!--More-->

+ 问题一、多个插件同时进行swizzling结果会怎么样   
+ 问题二、该以什么样的形式供项目使用
# 一、Method Swizzling原理
## 1.1、Runtime   
+ 首先，看一下Class的结构      

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
## 1.2、Category VS 自定类

对一个类进行Swizzling有两种方法，一种实现这个类的`Category`；另外一种就是自定一个类。但是这两种方式还是有一定的区别的。  

+ Category    
Category方法定义的方法都自然而然地添加到原来类了，特别是需要实现某些第三Delegate时比较方便。    
+ 自定义类    
其实Swizzling本质是将两个方法的实现进行交换，它的context应该是相同。所以一般会将自定类的实现先动态添加至需要被交换的类中



## 1.3、注意点

+ Swizzling最好在`+load`中做， 关于`+load`与`+initialize`的区别可以参考[Objective-C +load vs +initialize](http://blog.leichunfeng.com/blog/2015/05/02/objective-c-plus-load-vs-plus-initialize/)
+ 使用`dispatch_once`处理Swizzling部分代码，以保证代码只会执行一次且线程安全。  
+ Swizzle后记得在交换的方法里调用被交换的方法。

# 二、Method Swizzling实现    

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
> 注释的部分是自定类的处理部分，因为类的Category会自动将`swizzled`方法添加至类中

# 三、Swizzling 与 load+Notification
## 4.1 load+Notification
+ load+Notification**监听方法需要是类方法**，另外需要注意的是**在OC中类方法中`self`为`Class`**与`Object`不一样。因为`load`是类方法则监听方法必须为类方法。
+ 基于上一点，如果需要实现第三Delegate时，是无法满足需求的。因为一般Delegate方法都不会是类方法    

# 四、示例
# 五、展望
Swizzling称为iOS的黑魔法，利用得当会给项目带很多好处，但是如果不当也可能会造成灾难性问题。因些如果要用好些方法不仅仅是会写上面的代码还需要在联调迅速解决问题，因些这就需要对其本质做更多的探索，只有对它了如指掌才能发挥其最大潜力。