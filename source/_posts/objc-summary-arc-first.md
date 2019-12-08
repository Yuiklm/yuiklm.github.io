---
layout: post
title: 【《Objective-C 高级编程 iOS 与 OS X 多线程与内存管理 》 阅读系列 】之 自动引用计数篇（一）
draft: false
date: 2018-06-09 21:11:43
categories:
tags: Objective-C,ARC,Autorelease
permalink:
description:
cover_img:
toc-disable:
comments:
---

![](http://wx2.sinaimg.cn/mw690/699458aegy1fs591c4724j21590p0u0x.jpg)

> #### 导读
> 本系列，作为 《Objective-C 高级编程 iOS 与 OS X 多线程与内存管理 》一书的阅读总结，内容以原书内容提炼 + 内容扩展+个人理解，但大部分还是以提炼书中内容为主，以保证内容正确性。  
> 
> 相信大部分 iOS 开发者都对 **自动引用计数** 不陌生，**自动引用计数（ARC，Automatic Reference Counting）**是指内存管理中对引用采取自动计数的技术，官方文档说的是：“**在 LVVM 编译器中设置 ARC 为有效状态，就无需再次键入 retain 或者是 release 代码**”。
> 
> 在介绍 ARC  之前，我们先了解什么是引用计数，也就是 Objective-C 中的内存管理。

<!-- more -->

#### 1. 引用计数

> 我们都知道，在 MRC 中，当对象的被创建并被持有时，对象的引用计数为 1 ，执行 `[id retain]` 对象的引用计数递增，执行 `[id release]` 对象的引用计数递减，当对象的引用计数递减到 0 时，对象被销毁。

以上为 iOS 开发者所熟知的 Objective-C 内存管理方式，下面简单介绍 Objective-C 如何通过引用计数来控制对象的生命周期。

对于 `64` 位环境（ iPhone5s 之后的机型）来说，对象中指向一个 `isa_t` 结构体的 isa 指针包含类的各种信息，“一些”对象的`isa_t` 会拿出一部分空间存储引用计数，会有 19 位来存储引用计数。  
对于较老机型 `19bit` 环境保存引用计数不一定够，那么这个时候引用计数会由专门的 SideTable 类来存储。

**`isa_t` 结构体** 

	struct objc_object {
	    isa_t isa;
	};

	//
	
	union isa_t  {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;


	   struct {
	        uintptr_t indexed           : 1;
	        uintptr_t has_assoc         : 1;
	        uintptr_t has_cxx_dtor      : 1;
	        uintptr_t shiftcls          : 30; // MACH_VM_MAX_ADDRESS 0x1a0000000
	        uintptr_t magic             : 9;
	        uintptr_t weakly_referenced : 1;
	        uintptr_t deallocating      : 1;
	        uintptr_t has_sidetable_rc  : 1;
	        uintptr_t extra_rc          : 19;
		    define RC_ONE   (1ULL<<45)
	        define RC_HALF  (1ULL<<18)
	    };
	    
可以看出， isa 指向的结构体包含对象的各种信息，其中几个与引用计数有关的变量：

| 变量 | 含义 |
| :-------------: |:-------------:| 
| indexed | 0 表示普通的 isa 指针，1 表示使用优化，存储引用计数 | 
| has_sidetable_rc | 表示该对象的引用计数值是否过大无法存储在 isa 指针 | 
| extra_rc | 存储引用计数值减一后的结果 |


**SideTable 类 用来管理引用计数表和 weak 表**

```
spinlock_t slock;	//保证原子操作的自选锁
RefcountMap refcnts;	//保存引用计数的散列表
weak_table_t weak_table; //保存 weak 引用的全局散列表
```

#### 2. 内存的思考方式

书中介绍，对于计数式内存管理的客观、正确的思考方式是：

1. 自己生成的对象，自己所持有
2. 非自己生成的对象，自己也能持有
3. 不再需要自己持有的对象时释放
4. 非自己持有的对象无法释放

对象操作与 Objective-C 方法

| 对象操作 | Objective-C 方法 |
| :-------------: |:-------------:| 
| 生成并持有对象 | alloc/new/copy/mutableCopy 等方法 | 
| 持有对象 | retain 方法 | 
| 释放对象 | release 方法 |
| 废弃对象 | dealloc 方法 |

思考方式除第 2 点外，其他都好理解，着重看下第 2 点，对于使用 `alloc` `new` `copy` `mutableCopy` 生成的对象，我们认为是自己生成的对象，且自己持有，那么什么是“非自己生成的对象”？

```  
- (id)allocObject
{
	/** 自己生成并持有对象 */
	id obj = [[NSObject alloc] init];
	return obj;
}

```

```
/** 取得非自己生成并持有的对象 */
id obj1 = [obj0 allObject];
```

以上是 “非自己生成的对象并持有”，那么“非自己生成的对象自己又不持有呢”？  

MRC
 
```
- (id)object
{
	id obj = [[NSObject alloc] init];
	[obj autorelease];
	/** 取得对象存在，但自己不持有 */
	return obj;
}
```
```
id obj1 = [obj0 object];
/** 取得非自己生成且不持有的对象 */
[obj1 retain];
/** 自己持有对象 */
```

使用 `autorelease` 方法，使得取得的对象存在，但自己不持有对象，因为该对象将在超出指定的生存范围时能够自动并正确地释放（调用 `release` 方法），`autorelease` 方法会像 c 语言的自动变量那样来对待对象实例，当超出其作用域（相当于变量作用域）时，对象实例的 `release`实例方法被调用，且**可以设定对象实例的作用域**

`autorelease` 的具体使用方法：

1. 生成并持有 NSAutoreleasePool 对象
2. 调用已分配对象的 `autorelease` 实例方法
3. 废弃 NSAutoreleasePool 对象

