---
title: C++标准Attribute
layout: post
tags:
- c++
- attribute
date: 2018-08-26
excerpt: 列举了至c++17标准定义的attribute
---

虽然这些特性估计八辈子是用不上了，不过还是记录一下：

c++11开始定义标准attribute，用以控制编译器的某些行为；此前用于这个目的的有**#pragma**,**__attribute__**,**__declspec**等

截至到目前的c++17标准，定义的并不多，如下：

| 标识符                   | 用途                                                                               |
| ------                   | ----                                                                               |
| [[noreturn]]             | 有此标识的函数声明表明该函数不返回（如果有返回，UD）                               |
| [[carries_dependency]]   | 与多线程下内存优化有关                                                             |
| [[deprecated("reason")]] | 有此标识的标识符在被使用时发出编译器警告，但可以照常使用                           |
| [[fallthrough]]          | 有此标识的switch语句的case段在执行后没有跳过之后的case段，编译器不会发出警告       |
| [[nodiscard]]            | 有此标识的函数或者返回有此标识的枚举、类对象的函数在返回值被丢弃时，编译器发出警告 |
| [[maybe_unused]]         | 有此标识的标识符未被使用，编译器不会警告                                           |

根据[这篇博客](https://akrzemi1.wordpress.com/2018/01/24/help-the-compiler-warn-you/),这些attribute可以用于控制编译器的警告，规避潜在的问题。比如[[nodiscard]]就适用于有bool返回值标识函数运行成功与否的情况。在这种情况下，调用者如果没有用if检查返回值，编译器会发出警告（当然，使用者可以用很简单的hack来规避警告，不过这种刻意的行为毫无意义）
