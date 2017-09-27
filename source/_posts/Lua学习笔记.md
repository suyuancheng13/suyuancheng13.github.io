---
title: Lua学习笔记
date: 2017-07-10 17:06:34
tags: [Lua, Dev]
categories: Lua
---

> Lua 学习笔记

<!--More-->

# 一、基础知识
## 1.1、 Lua源码学习
### 1.1.1、C语言函数名与宏定义冲突问题
所望的语言函数名与宏定义冲突问题，就是指宏定义与函数相同时编译器不知道标识符该怎么处理。如下代码：

	#pragma once

	#include <stdio.h>
	#define f() printf("this is define ");
	void f();

上述代码在VS2010会检测出`Error:不允许使用不完整定义`。解决方案很简单，如下：

	#pragma once

	#include <stdio.h>
	#define f() printf("this is define ");
	void (f)();

在函数名上添加小括号即可解决问题。这也就是为什么lua.h头文件里为什么那么函数名都带括号的原因:

	/*
	** access functions (stack -> C)
	*/
	
	LUA_API int             (lua_isnumber) (lua_State *L, int idx);
	LUA_API int             (lua_isstring) (lua_State *L, int idx);
	LUA_API int             (lua_iscfunction) (lua_State *L, int idx);
	LUA_API int             (lua_isinteger) (lua_State *L, int idx);
	LUA_API int             (lua_isuserdata) (lua_State *L, int idx);
	LUA_API int             (lua_type) (lua_State *L, int idx);
	LUA_API const char     *(lua_typename) (lua_State *L, int tp);
	
	LUA_API lua_Number      (lua_tonumberx) (lua_State *L, int idx, int *isnum);
	LUA_API lua_Integer     (lua_tointegerx) (lua_State *L, int idx, int *isnum);
	LUA_API int             (lua_toboolean) (lua_State *L, int idx);
	LUA_API const char     *(lua_tolstring) (lua_State *L, int idx, size_t *len);
	LUA_API size_t          (lua_rawlen) (lua_State *L, int idx);
	LUA_API lua_CFunction   (lua_tocfunction) (lua_State *L, int idx);
	LUA_API void	       *(lua_touserdata) (lua_State *L, int idx);
	LUA_API lua_State      *(lua_tothread) (lua_State *L, int idx);
	LUA_API const void     *(lua_topointer) (lua_State *L, int idx);
	....

### 1.1.2、C语言大量使用宏定义
+ 宏定义函数，减少函数调用开销
+ 宏定义方便    

## 2、Lua语法
### 2.1、Lua函数调用方法`.`与`:`区别    
`.`:是一般的调用     
`:`:自带第一个参数`self`