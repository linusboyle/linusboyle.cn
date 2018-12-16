---
title: 关于不完全的类型
date: 2018-07-20
tags:
- sizeof
- c++
- 笔记
layout: post
excerpt: 一段很短的笔记，关于编译期的类型
---

今天看了看qt里QScopedPointer的实现，类本身很简单，需要注意的地方是由于这是一个智能指针，在创建时需要分配内存（什么？可以是空的？没什么用……）

如果代理的指针属于一个前置声明的类型，那么需要注意的是，在使用到智能指针成员的地方必须有前置类的定义。

换句话说，只有在实现文件里定义构造、析构、赋值等。

这里qt判断不完全类型的方法是sizeof运算符（其实不用也可以，编译器可以发现这个问题。不过在网上看到一些资料，部分编译器会让这种不完全的类型编过，所以这里用sizeof显式地要求还是合理一些）

```cpp
template <typename T>
struct QScopedPointerDeleter
{
    static inline void cleanup(T *pointer)
    {
        // Enforce a complete type.
        // If you get a compile error here, read the section on forward declared
        // classes in the QScopedPointer documentation.
        typedef char IsIncompleteType[ sizeof(T) ? 1 : -1 ];
        (void) sizeof(IsIncompleteType);

        delete pointer;
    }
};
```

用到前置声明的话，对于类型的定义是不清楚的，所以在头文件里定义类的构造、析构、赋值是当然不行的。
