---
title: Blocks in iOS
date: 2017-03-19 11:01:20
categories: iOS
tags: [iOS, Dev]
---
> 最近由于工作原因，对iOS Block进行一次学习,本篇文章对Block的内存使用相关的内容简要整理一下，解释其中一些原理和使用Block需要注意的问题 。

<!--More-->

# 一、关于Block的内存表示
## 1.1、`clang -rewite-objc 源文件 -o 目标文件（可以为.txt）`
+ 首先我们来看一段常见源码，通过上述命令查看其中间文件：

```     

	typedef void (^ test1) (BOOL success, NSError *error) ;

	int main(int argc, char * argv[]) {
	    @autoreleasepool {
	        int base =10;
	        for(int i=0;i<10;i++){
	            int a =10;
	            // int *b  = (int*)malloc(10*sizeof(int));
	            void (^ test) (BOOL success, NSError *error)= ^(BOOL success, NSError *error) {
	                NSLog(@"sssss:%d,base:%d",success,base);
	            } ;
	            test(1,nil);
	           
	        }
	
	//        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
	    }
	}

```

+ 通过sublime打开目标文件会发现代码远不止上面那寥寥几行，不难从其中可以找到如下与Block相关的代码。

```

	struct __block_impl {
  		void *isa;
  		int Flags;
  		int Reserved;
  		void *FuncPtr;
	};
```

`struct __block_impl`就是block的真面目，结构还是相当清晰。`isa`对OC开发人员再熟悉不过，是表示其类型或者说本质的，block有三种类型`NSGlobalBlock`，`NSStackBlock`,`NSMallocBlock`，不同类型的block就是通过这个字段区分。然后就是`FuncPtr`，顾名思义就是指向某一函数的指针。其实block本质上与c语言中的函数指针是一样的。

```   

	struct __main_block_impl_0 {
	  struct __block_impl impl;
	  struct __main_block_desc_0* Desc;
	  int base;
	  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _base, int flags=0) : base(_base) {
	    impl.isa = &_NSConcreteStackBlock;
	    impl.Flags = flags;
	    impl.FuncPtr = fp;
	    Desc = desc;
	  }
	};
	
	static void __main_block_func_0(struct __main_block_impl_0 *__cself, BOOL success, NSError *error) {
	  int base = __cself->base; // bound by copy
	
	                NSLog((NSString *)&__NSConstantStringImpl__var_folders_rl_k02s7ny56tj7m7cg3kdrfpbh0000gn_T_main_0c0cda_mi_0,success,base);
	            }

	static struct __main_block_desc_0 {
	  size_t reserved;
	  size_t Block_size;
	} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
```
```

	int main(int argc, char * argv[]) {
	    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
	        int base =10;
	
	        for(int i=0;i<10;i++){
	            int a =10;
	
	            void (* test) (BOOL success, NSError *error)= (void (*)(BOOL, NSError *))((id (*)(id, SEL))(void *)objc_msgSend)((id)((void (*)(BOOL, NSError *))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, base)), sel_registerName("copy"));
	            ((void (*)(__block_impl *, BOOL, NSError *))((__block_impl *)test)->FuncPtr)((__block_impl *)test, 1, __null);
	
	    }
	}
```
以上这一段代码正是与源码相对应的`struct __main_block_impl_0`是此处block的真正实现，主要分为两部分一个是block的定义一个是block的一些描述。`__main_block_func_0`是具体执行的函数体。接下来就是要`main`函数体中初始化block，然后就大功告成可以便可享用block。
以上揭示了block的本质。
# 二、关于Block类型
# 三、__block变量
```

	typedef void (^ test1) (BOOL success, NSError *error) ;

	int main(int argc, char * argv[]) {
	    @autoreleasepool {
	        int base =10;
	        for(int i=0;i<10;i++){
	            int a =10;
	            // int *b  = (int*)malloc(10*sizeof(int));
	            void (^ test) (BOOL success, NSError *error)= ^(BOOL success, NSError *error) {
	                NSLog(@"sssss:%d,base:%d",success,base);
	            } ;
	            test(1,nil);
	           
	        }
	
	//        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
	    }
	}

```


```

	struct __Block_byref_base_0 {
	  void *__isa;
	__Block_byref_base_0 *__forwarding;
	 int __flags;
	 int __size;
	 int base;
	};

	struct __main_block_impl_0 {
	  struct __block_impl impl;
	  struct __main_block_desc_0* Desc;
	  __Block_byref_base_0 *base; // by ref
	  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_base_0 *_base, int flags=0) : base(_base->__forwarding) {
	    impl.isa = &_NSConcreteStackBlock;
	    impl.Flags = flags;
	    impl.FuncPtr = fp;
	    Desc = desc;
	  }
	};
	static void __main_block_func_0(struct __main_block_impl_0 *__cself, BOOL success, NSError *error) {
	  __Block_byref_base_0 *base = __cself->base; // bound by ref
	
	                NSLog((NSString *)&__NSConstantStringImpl__var_folders_rl_k02s7ny56tj7m7cg3kdrfpbh0000gn_T_main_5fdc20_mi_0,success,(base->__forwarding->base));
	            }
	static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->base, (void*)src->base, 8/*BLOCK_FIELD_IS_BYREF*/);}
	
	static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->base, 8/*BLOCK_FIELD_IS_BYREF*/);}
	
	static struct __main_block_desc_0 {
	  size_t reserved;
	  size_t Block_size;
	  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
	  void (*dispose)(struct __main_block_impl_0*);
	} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
	int main(int argc, char * argv[]) {
	    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
	        __attribute__((__blocks__(byref))) __Block_byref_base_0 base = {(void*)0,(__Block_byref_base_0 *)&base, 0, sizeof(__Block_byref_base_0), 10};
	        for(int i=0;i<10;i++){
	            int a =10;
	
	            void (* test) (BOOL success, NSError *error)= ((void (*)(BOOL, NSError *))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_base_0 *)&base, 570425344)) ;
	            ((void (*)(__block_impl *, BOOL, NSError *))((__block_impl *)test)->FuncPtr)((__block_impl *)test, 1, __null);
	
	        }
	
	
	    }
	}
```
