---
title: c++对象模型
date: 2018-07-04
layout: post
tags: c++ object model
---
本文算是[Richard Powell的"Intro to the C++ Object Model"](https://www.youtube.com/watch?v=iLiDezv_Frk)的观后感和笔记，建议大家看一看这个演讲，让人受益匪浅。

## POD
POD即plain old data,在c++中代表和ansi-c兼容，可以直接当作c结构体作用于c标准库的对象类型。这种对象可以用malloc分配内存、memmove拷贝等。

使用std::is_pod可以检测是否是POD类型，简单做一个测试：

```cpp
#include <type_traits>
#include <iostream>

struct grid{
    int x;
    int y;
};

struct integer{
    int getvalue(){return x;}
    private:
        int x;
};

struct foo{
    int x;
    private:
    int y;
};

int main(){
    std::cout<<std::boolalpha;
    std::cout<<std::is_pod<grid>::value<<std::endl;
    std::cout<<std::is_pod<integer>::value<<std::endl;
    std::cout<<std::is_pod<foo>::value<<std::endl;
}
```

输出是true true false

可见成员函数并不影响对象实例是否是pod类型，而且简单的sizeof大小检测也表明普通的方法不改变类实例的大小,这和下节所讲的编译器行为有关

（不同权限的对象和虚函数都会导致non-pod，不过这和编译器实现方式有关，c++标准并没有定义这些情况下对象应该按照何种方式对齐）

## 成员函数
c++的成员函数并不是通过在实例里包含函数指针实现的（这种实现方法很浪费），成员函数实际上类似于全局函数

假设有这么一个函数原型,它将被转换为下面的实际函数类型：

```cpp
float Complex::Abs() const

//converted to function below

float _ZNK7Complex3AbsEv(Complex const * this)
```
* 首先，编译器会将成员函数的第一个参数隐式的变为this指针
* 所有函数定义中的成员变量都变为显式的this->形式
* 将函数const类型标识符改加到this上
* hash函数名，使其和类无关；

(注：在c++和java里，这个过程叫**mangling**，碾压？？)

调用函数时，类似的变换：
```cpp
Complex foo;
float bar=foo.Abs();

//converted to ...

float bar=_ZNK7Complex3AbsEv(&foo);
```

当你用nm命令查看一个可执行文件时，出现的一大堆_Z开头的标识符其实就是这类经过编译器处理后的名称，当然其中不仅仅包含有成员函数(比如函数重载等)。如果要把一个字符串转换回实际的c++类型，可以用**c++filt**命令,详见manpage

## 虚函数
是的，关于虚函数，很多人已经解释地很清楚，不过我这里还是要说明一下。

假设类中有一个虚函数virtual void foo();和上一节提到的函数一样，它会被转换为全局的函数，并且经历一系列编译器的操作。普通成员函数和虚函数的区别在于，普通函数在编译期即能绑定，而虚函数则在运行期。

回忆上面提到的调用函数时所做的操作，很显然普通函数如果出现在继承中被重写隐藏的情况，那么调用哪一个函数和调用函数的对象的字面类型确定。如果是虚函数呢？

拥有虚函数的类，其实例的大小大于成员大小的和，因为建立了指向虚函数表的指针。虚函数表中是指实际函数的指针，不同类的虚函数表成员指向不同的函数。在对象实例化时虚指针正确地指向虚函数表，从而在调用时实现多态。控制虚指针指向正确的虚函数表的是类的构造函数——所以很明显，构造函数不能是虚函数。

最后注一句：c++11标准之后，应该尽量使用override和final
