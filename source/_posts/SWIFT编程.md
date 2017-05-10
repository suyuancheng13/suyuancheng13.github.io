---
title: SWIFT编程
date: 2017-05-07 20:18:41
tags: [iOS, swift]
categories: iOS
---
    

> Swift学习与编程笔记

<!--More-->    

# 一、基础知识

## 1.1 Swift单例

原理都是通过类属性实现，访问时通过类直接获取。

		//MARK: 单例2
		let instance = Singleton()
		class Singleton {
			//MARK: 单例1
			static let sharedInstance = Singleton()
		
			//MARK: 单例2 
			class var sharedInstance2: Singleton {
				return instance
			}
		
			//MARK: 单例3
			static var shareInstance3:Single{
		        struct MyStatic{
		            static var instance :Single = Single()
		        }
		        return MyStatic.instance;
		    }
		}

**Caution:跨模块（framework）使用时Xcode不会提示相应类属性，手动写上即可；另外，`let`是线程安全的**

# 二、Swift混编
## 2.1 Swift 与 OC 混编
### 2.1.1 OC调用Swift
OC要调用Swift需要借助于`yourtargetname-swift.h`   
 
+ 当我们在一个OC的工程中添加Swift语言Xcode会提示是否添加桥接文件；
+ 创建Swift类，为了让OC能调用还需要在类前添加`@objc`表示Swift代码暴露给OC用；
+ 但OC怎么使用Swift呢？这就需要`yourtargetname-swift.h`，这个头文件名可以要`target->Build Setting->Objective-C Generated Interface Header Name`。**需要注意的是这个头文件target中只有一个，而且是看不到的，它是会在编译时生成**，代码如下：

		#import "yourtargetname-swift.h"
		
		...		
		swiftclass *instance = [[swiftclass alloc]init];
	
### 2.1.2 Swift调用OC

+ Swift要调用OC需要借助于`yourproject-Bridging-Header.h`,在这个头文件里通过`import`相关OC库头文件，如：

		#import <GoogleMaps/GoogleMaps.h>


**注意：**   

+ **这个头文件Xcode会帮你生成，只要你在Swift工程添加OC类Xcode就会提示你是否需要生成这个文件。**
+ **如果一个是在工程中创建了一个framework的target，此时系统会生成一个`yourtargetname.h`的文件，在这个头文件里通过`import`相关OC库头文件即可。**