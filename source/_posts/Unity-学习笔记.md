---
title: Unity 学习笔记
date: 2017-06-27 15:23:37
tags: [Unity, Dev]
categories: Unity
---
> Unity 学习笔记(包括C#)

<!--More-->

# 一、基础
## 1.1 Gameobject

Gameobject是游戏中最重要的一个角色，游戏中任务一个对象都是一个Gameobject。
## 1.2 prefab

预设 (Prefabs) 是可重复用于场景中的游戏对象 (GameObject) 和组件 (Component) 的集合。一个预设 (Prefab) 可创建多个相同对象，称之为实例化。Prefab可以理解为结构蓝图，如果蓝图变了实例也会跟着一起变。
## 1.3 Shader，Material，Texture
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