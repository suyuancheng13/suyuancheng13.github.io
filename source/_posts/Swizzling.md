layout: method
title: Swizzling
date: 2017-07-25 15:06:57
tags: [iOS, Dev]
categories: iOS
---

> iOS Method　Swizzling学习笔记    

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

源码中可以很容易知道在Objective C中的一个方法，其实最终会转换成`objc_method`的结构体，而结构体中清晰可以看到`SEL`与`IMP`。简而言之，就是一个OC的类有一个`SEL`表示函数名，`IMP`（函数指针）指向函数的具体实现。调用一个函数时便是通过`SEL`找到对应的`IMP`。所谓的Swizzling就是交换两个方法的`IMP`。
## 1.2、Category VS 自定类

对一个类进行Swizzling有两种方法，一种实现这个类的`Category`；另外一种就是自定一个类。但是这两种方式还是有一定的区别的。  

+ Category    
Category方法定义的方法都自然而然地添加到原来类了，特别是需要实现某些第三Delegate时比较方便。

## 1.3、注意点

+ Swizzling最好在`+load`中做， 关于`+load`与`+initialize`的区别可以参考[Objective-C +load vs +initialize](http://blog.leichunfeng.com/blog/2015/05/02/objective-c-plus-load-vs-plus-initialize/)
+ 使用`dispatch_once`处理Swizzling部分代码，防止子类影响父类   
+ 
# 二、Method Swizzling实现    

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


# 三、项目怎么样方便使用