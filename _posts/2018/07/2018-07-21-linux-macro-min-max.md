---
date: 2018-07-21
layout: post
title: 关于Linux中min/max宏写法
tags:
- linux
- macro
- 宏
excerpt: 一小段关于linux源码中min和max的正确定义的讨论
---

[这篇博客](http://gaomf.cn/2017/10/08/Kernel_min_max_macro/)和以往的对linux源码宏定义的解释不太一样，它所讲解的是这样一段定义：

```c
#define __min(t1, t2, min1, min2, x, y) ({              \
	t1 min1 = (x);                                  \
	t2 min2 = (y);                                  \
	(void) (&min1 == &min2);                        \
	min1 < min2 ? min1 : min2; })

#define ___PASTE(a,b) a##b
#define __PASTE(a,b) ___PASTE(a,b)

#define __UNIQUE_ID(prefix) __PASTE(__PASTE(__UNIQUE_ID_, prefix), __COUNTER__)

#define min(x, y)                                       \
	__min(typeof(x), typeof(y),                     \
	      __UNIQUE_ID(min1_), __UNIQUE_ID(min2_),   \
	      x, y)

#define min_t(type, x, y)                               \
	__min(type, type,                               \
	      __UNIQUE_ID(min1_), __UNIQUE_ID(min2_),   \
	      x, y)
```
详见Linux源码：tools/testing/scatterlist/linux/mm.h

我大致搜索了一下源码，这一部分并不是真正使用的定义(就我所发现，似乎这个头文件没有被包含过)

一般来说的定义是：
```c
#ifndef max
#define max(x, y) ({				\
	typeof(x) _max1 = (x);			\
	typeof(y) _max2 = (y);			\
	(void) (&_max1 == &_max2);		\
	_max1 > _max2 ? _max1 : _max2; })
#endif

#ifndef min
#define min(x, y) ({				\
	typeof(x) _min1 = (x);			\
	typeof(y) _min2 = (y);			\
	(void) (&_min1 == &_min2);		\
	_min1 < _min2 ? _min1 : _min2; })
#endif
```

定义在tools/include/linux/kernel.h
