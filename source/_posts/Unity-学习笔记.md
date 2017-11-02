---
title: Unity 学习笔记
date: 2017-06-27 15:23:37
tags: [Unity, Dev]
categories: Unity
---
> Unity 学习笔记(包括C#)

<!--More-->

# Reference
+ [Unity manualt翻译](https://nuysoft.gitbooks.io/unity-manual/content/)
+ [Unity mannual](https://docs.unity3d.com/Manual/index.html)
+ [Unity圣典](http://www.ceeger.com/forum/)
+ [Unity 蛮牛](http://www.manew.com/)    

# 一、基础
## 1.1 Gameobject

Gameobject是游戏中最重要的一个角色，游戏中任务一个对象都是一个Gameobject。
## 1.2 prefab

预设 (Prefabs) 是可重复用于场景中的游戏对象 (GameObject) 和组件 (Component) 的集合。一个预设 (Prefab) 可创建多个相同对象，称之为实例化。Prefab可以理解为结构蓝图，如果蓝图变了实例也会跟着一起变。

## 1.3 Shader，Material，Texture，Mesh


|unity| 物体 |     
|:--|:--|     
|Texture|表面呈现的图案，bitmap images|   
|Material|表示物体的材料与质感：光滑度，透明度，反射率，发光度等，由Texture+Shader|    
|[Shader](https://onevcat.com/2013/07/shader-tutorial-1/)|是一个脚本，将贴图，颜色，mesh进行mix，然后输出可绘制的单元|     
|Mesh|物体的外形|     



[参考](http://docs.manew.com/Manual/index.htm)      

着色器(Shader)可定义以下事项：

+ 渲染对象的方法。这包括使用不同方法，具体取决于最终用户的图形卡。
+ 用于渲染的任何顶点照明和部分照明程序。
+ 材质 (Materials) 内可供分配的纹理属性。
+ 材质 (Materials) 内可供分配的颜色和数量设置。    

材质 (Materials) 可定义：　　　

+ 渲染使用的纹理。　　
+ 渲染使用的颜色。　　
+ 任何其他资源，如着色器进行渲染所需的立方体贴图 (Cubemap)。　　　

着色器由图形程序员编写。程序员使用非常简单的 ShaderLab 语言创建着色器。但是，让着色器在各种不同图形卡上充分发挥其作用是一项复杂的工作，这需要对图形卡运行原理有非常全面地了解。

## 1.4 Unity线程，协程

### unity是一个单线程引擎，主要原因是UnityEngine相关SDK只能在主线程中跑 

协程：   

+ yield; 在下一帧上调用所有 Update 函数后，协同程序将继续运行。
+ yield WaitForSeconds(2); 在指定的时间延迟之后，为此帧调用所有 Update 函数之后继续运行
+ yield WaitForFixedUpdate(); 在所有脚本上调用所有 FixedUpdate 后继续运行
+ yield WWW 完成 WWW 下载后继续运行。
+ yield StartCoroutine(MyFunc); 连接协同程序，并等待 MyFunc coroutine 首先结束。

## 1.5 Unity Life Cycle

![img](https://docs.unity3d.com/uploads/Main/monobehaviour_flowchart.svg)  

## 1.6 Unity坐标系
![img](http://answers.unity3d.com/storage/temp/8053-spaces.jpg)   

## 1.7 iOS编译问题

从Unity导出iOS工程时需要注意是否使用了一些JIT(just in time compiler)特性的语法（例如C# Linq的排序）。因为iOS编译是AOT((Ahead-Of-Time，静态编译)不支持动态编译。

# 二、实践
## 2.1 Unity使用DLL（动态链接库）
### 2.1.1、 生成DLL

DLL的生成是需要借助于VS的，Unity自己是无法做到的。    

+ 新建C#**类库项目**，注意项目类型必须是类库项目

+ 编写代码，与正常开发无异
+ 将项目属性 -> 应用程序 -> 目标框架：改为` .NET Framework 3.5`或以下 。这一步很重要，因为Unity3D（当前的Unity3D版本是3.5版） 支持的 .Net 是3.5版。

然后run一下就会生成一个DLL库

### 2.1.2、Unity使用DLL
Unity使用DLL就相当简单了，只需要将DLL拖入Unity工程随便什么目录都可以，一般是放置到Plugin目录下。然后，C#就可以正常使用DLL中的代码。