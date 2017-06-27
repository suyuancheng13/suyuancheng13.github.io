---
title: Unity 学习笔记
date: 2017-06-27 15:23:37
tags: [Unity, Dev]
categories: Unity
---
> Unity 学习笔记(包括C#)

<!--More-->

# 一、基础
## 2.1 Unity使用DLL（动态链接库）
### 2.1.1、 生成DLL

DLL的生成是需要借助于VS的，Unity自己是无法做到的。    

+ 新建C#**类库项目**，注意项目类型必须是类库项目

+ 编写代码，与正常开发无异
+ 将项目属性 -> 应用程序 -> 目标框架：改为` .NET Framework 3.5`或以下 。这一步很重要，因为Unity3D（当前的Unity3D版本是3.5版） 支持的 .Net 是3.5版。

然后run一下就会生成一个DLL库

### 2.1.2、Unity使用DLL
Unity使用DLL就相当简单了，只需要将DLL拖入Unity工程随便什么目录都可以，一般是放置到Plugin目录下。然后，C#就可以正常使用DLL中的代码。