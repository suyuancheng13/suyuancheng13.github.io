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

## 1.2、构造函数
### 1.2.1 在构造函数里可以对常量进行修改，一旦修改后这个值就不会变了

		class test {
		    let text: String
		    init(text: String) {
		        self.text = text
		    }
		}


### 1.2.2 系统合成构造函数主要包括两种：默认构造函数（不用多说），逐一赋值构造函数（结构体，类类型是没有的）
				
		struct Size {
		    var width = 0.0, height = 0.0
		}
		let zhuyi = Size(width: 2.0, height: 2.0)

### 1.2.2 两段式构造
+ 第一个阶段，每个存储型属性被引入它们的类指定一个初始值。当每个存储型属性的初始值被确定后;  
+ 第二阶段开始，它给每个类一次机会，在新实例准备使用之前进一步定制它们的存储型属性。

### 1.2.3 指定构造器，便利构造器
+ **指定构造器**是类必须有的至少一个。可以通过继承了父类中的指定构造器而满足了这个条件

		class test {
			init(parametes)
		}

+ **便利构造器**是类中比较次要的、辅助型的构造器。  

		 class test {
			convenience  init(parametes)
			}   
两者的关系如下图：   
![img](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Art/initializerDelegation01_2x.png)     
**1.指定构造器必须向上也就是父类继续**   
**2.便利构造器必须横向向指定构造器依赖，也就是便利构造器必须导致一个指定构造器被调用**  


+ 便利构造器不能直接调用父类的便利构造器
+ 构造器的自动继承    
子类在默认情况下不会继承父类的构造器。但是如果满足特定条件，父类构造器是可以被自动继承的。
 	+ 子类中引入的所有新属性都提供了默认值
 	+ 如果子类没有定义任何**指定构造器**，它将自动继承所有父类的**指定构造器**。（OC继承；C++不继承）
 	+ 如果子类提供了所有父类**指定构造器**的实现——无论是通过规则 1 继承过来的，还是提供了自定义实现——它将自动继承所有父类的**便利构造器**。
 	+ 子类可以将父类的**指定构造器**重写为**便利构造器**
 	

+ demo:
 
		class Food {
		    var name: String
		    init(name: String) {//指定构造器
		        self.name = name
		    }
		    convenience init() {//便利构造器， 符合横向代理原则：调用指定构造器
		        self.init(name: "[Unnamed]")
		    }
		}
		
		//继承
		class RecipeIngredient: Food {
		    var quantity: Int//新增属性
		    init(name: String, quantity: Int) {
				//符合两段式构造第一阶段，给属性一个初始化值
		        self.quantity = quantity//应该不可以访问self
		        super.init(name: name)//还是处于第一阶段，初始化父类部分属性
				
		    }
			//子类的便利构造器与父类的指定构造器匹配，因些也算重写所以加上 override
		    override convenience init(name: String) {
		        self.init(name: name, quantity: 1)//调用指定构造器
		    }
			
			//由于子类实现了父类的所有指定构造器，所以子类便继承了父类的便利构造器
			//convenience init() {
			//		 self.init(name: "[Unnamed]")
			// }
		}
		
		//子类没有构造器，所以继承所有的父类构造器
		class ShoppingListItem: RecipeIngredient {
		    var purchased = false
		    var description: String {
		        var output = "\(quantity) x \(name)"
		        output += purchased ? " ✔" : " ✘"
		        return output
		    }
		}


## 1.3 函数 & 闭包
### 1.3.1 函数
+ 引用形参`inout`,作用与C中的引用是一样的作用，然后在调用时使用`&`传值
+ 函数类型（闭包），swift中函数也是一种类型，就是通过参数列表及返回值来定义一个函数类型的，例如：
	
		var myFunc: (Int, Int)-> Int
+ 函数形参与标签，如果函数定义时没有标签时形参名就是标签   

		func test(number:Int, number1:Int)->Void

### 1.3.2 闭包其实就是代码块，与OC中的block类似。闭包表达式语法：

	{ (parametees) -> returnType in
		//statements
		
	}

闭包的函数体由关键字`in`引入

## 1.4 类型转换，检查
### 1.4.1 swift类型转主要通过两个操作符：`as?`与`as!`。`as?`表示转换时可能失败返回nil,`as!`类型转换表示一定会转换成功并强解引用
### 1.4.2 swift类型检查操作符：`is`。

# 二、Swift混编
## 2.1 Swift 与 OC 混编
### 2.1.1 OC调用Swift
OC要调用Swift需要借助于`yourtargetname-swift.h`   
 
+ 当我们在一个OC的工程中添加Swift语言Xcode会提示是否添加桥接文件；
+ 创建Swift类，为了让OC能调用还需要在类前添加`@objc`表示Swift代码暴露给OC用；***注意：`@objc`是不可以将非继承`NSObject`转换成OC可调用的对象***
+ 但OC怎么使用Swift呢？这就需要`yourtargetname-swift.h`，这个头文件名可以要`target->Build Setting->Objective-C Generated Interface Header Name`。**需要注意的是这个头文件target中只有一个，而且是看不到的，它是会在编译时生成，如果模块是一个framework可能在生成的framework头文件里可以看到此文件**，代码如下：

		#import "yourtargetname-swift.h"
		
		...		
		swiftclass *instance = [[swiftclass alloc]init];
	
### 2.1.2 Swift调用OC

+ Swift要调用OC需要借助于`yourproject-Bridging-Header.h`,在这个头文件里通过`import`相关OC库头文件，如：

		#import <GoogleMaps/GoogleMaps.h>


**注意：**   

+ **这个头文件Xcode会帮你生成，只要你在Swift工程添加OC类Xcode就会提示你是否需要生成这个文件。**
+ **如果一个是在工程中创建了一个framework的target，此时系统会生成一个`yourtargetname.h`的文件，在这个头文件里通过`import`相关OC库头文件即可。**