---
title: 单线程固定大小内存池&&C++内存分配的初步理解
date: 2018-07-01
layout: post
tags: c++ 内存池 内存机制
---

### 关键字new、delete的展开

new和delete是c++的关键字，只不过和其他关键字不同，使用这两个词需必须实现特定的运算符重载。一般来说我们给一个类的对象分配或回收内存会这么写：

```cpp
Foo* foo=new Foo(bar);
Foo* foo2=new Foo[count];

delete foo;
delete[] foo2;
```

上述语句由c++语法规定，被编译器展开为下述形式：

```cpp
void* tmp=operator new(sizeof(Foo));
tmp->Foo(bar);//这里的意思是在这块内存空间调用构造函数实际生成对象，伪代码写法
Foo* foo=(Foo*)tmp;

void* tmp2=operator new[](count*sizeof(Foo)+overhead);
void* ptr=tmp2;
ptr->Foo();
ptr+=sizeof(Foo);
...;//构建一系列对象
Foo* foo2=(Foo*)tmp2;

foo->~Foo();
operator delete(foo);

ptr=foo2;
ptr->~Foo();
ptr+=sizeof(Foo);
...;//全部析构
operator delete[](foo2);
```

也就是说，应用new和delete关键字，会隐式调用operator new和operator delete（new[]、delete[]实际上是在调用new和delete分配内存，只是之后的构建对象的方式不太一样）。在c++标准库实现了全局的new和delete运算符（运算符和关键字是不同的）。如果有一段程序没有使用标准库，那么也无法通过new和delete动态分配内存，和C不使用malloc/free是一样的。

clang的实现如下：

```cpp
void *
operator new(std::size_t size) _THROW_BAD_ALLOC
{
    if (size == 0)
        size = 1;
    void* p;
    while ((p = ::malloc(size)) == 0)
    {
        // If malloc fails and there is a new_handler,
        // call it to try free up memory.
        std::new_handler nh = std::get_new_handler();
        if (nh)
            nh();
        else
#ifndef _LIBCPP_NO_EXCEPTIONS
            throw std::bad_alloc();
#else
            break;
#endif
    }
    return p;
}

_LIBCPP_WEAK
void
operator delete(void* ptr) _NOEXCEPT
{
    if (ptr)
        ::free(ptr);
}
```

这是全局函数，并不在std命名空间里。因为不是通过模板写的，所以在libc++或者libstdc++中以二进制文件提供。一般程序所用到的就是全局的运算符重载，从本质上来说还是c的malloc/free，只是针对c++面向对象的特性，自动化地调用构造函数、析构函数，在内存正确地创建或释放对象。STL的内存分配器中的一种就是对全局operator new/delete的包装（也有一种对malloc/free的包装）。

### 单线程固定大小内存池

全局的重载需要考虑各种情况，并行、对齐、异常处理等等。很多这些特性并不是必要的，这里介绍通过重载类的operator new/delete 实现更快速高效的内存分配。先尝试对单一类设计单线程内存池，因为只对一个特定类有效，其分配的内存当然是固定大小的。

考虑复数类：

```cpp
class Complex{
  private:
  	double _a;//a+bi
  	double _b;
  public:
  	Complex(double a,double b):_a(a),_b(b){};
  	const Complex& operator*(const Complex& other){
    	return Complex(_a*other._a-_b*other._b,_a*other._b+_b*other._a);
  	}
	friend ostream& operator<<(ostream&,const Complex&);
  	friend istream& operator>>(istream&,Complex&);
};

ostream& operator<<(ostream& out,const Complex& from){
	out<<from._a<<"+"<<from._b<<"i";
    return out;
}
istream& operator>>(istream& in,Complex& to){ 
  	in>>to._a>>to._b;
  	return in;
}
```

现在要为这个单一类设计内存池，简单的做法是给类添加静态方法和成员，组成内存池，这样不需要单独的内存池类的功能。写一个辅助类链表完成：

```cpp
#include <iostream>

class NextInList{
  public:
  	NextInList* next;
};

class Complex{
  private:
  	double _a;//a+bi
  	double _b;
  	static void expand(){
    	size_t size=sizeof(Complex)>sizeof(NextInList)?sizeof(Complex):sizeof(NextInList);
      	
      	NextInList* ptr=static_cast<NextInList*>(static_cast<void*>(new char[size]));
      	for(int i=0;i<EXPANSION_SIZE;++i){ 
        	ptr->next=static_cast<NextInList*>(static_cast<void*>(new char[size]));
          	ptr=ptr->next;
        }
      	ptr->next=nullptr;
  	}
  	
  	enum{
		EXPANSION_SIZE=32;
    }
  public:
  	Complex(double a,double b):_a(a),_b(b){};
  	const Complex& operator*(const Complex& other){
    	return Complex(_a*other._a-_b*other._b,_a*other._b+_b*other._a);
  	}
	friend ostream& operator<<(ostream&,const Complex&);
  	friend istream& operator>>(istream&,Complex&);
  
  	static NextInList* list;
  	static void newMemPool(){
    	expand();
  	}
  	static void delMemPool(){
    	NextInList* ptr=list;
      	while(ptr){
        	list=list->next;
          	delete[] ptr;
          	ptr=list;
      	}
  	}
  	inline void* operator new(size_t size){
    	if(list==nullptr){ 
        	expand();
        }
      	
      	NextInList* head=list;
      	list=list->next;
      	
      	return head;
  	}
  	inline void operator delete(void* doomed,size_t size){ 
    	NextInList* head=static_cast<NextInList*>(doomed);
      	head->next=list;
      	list=head;
    }
};

ostream& operator<<(ostream& out,const Complex& from){
	out<<from._a<<"+"<<from._b<<"i";
    return out;
}

istream& operator>>(istream& in,Complex& to){ 
  	in>>to._a>>to._b;
  	return in;
}

Complex::list=nullptr;
```

测试效率：

```cpp
#include <boost/timer>
#include <iostream>
#include "Complex.h"

int main(){
	Complex* arr[1000];
  	boost::timer timer;
  	for(int i=0;i<500;i++){
    	for(int j=0;j<1000;j++){ 
        	arr[j]=new Complex(i,j);
        }
      	for(int j=0;j<1000;j++){ 
        	delete arr[j];
        }
  	}
  	std::cout<<timer.elapsed()<<std::endl;
}
```

这个内存池只能增大，不能缩小，应付这种工作倒是足够了，不过这种实现方法并不是特别优雅。粗略更改数组大小可以得知，在数据量较大的时候内存池的优势比较明显。这种实现方法是《提高c++性能的编程艺术》这本书使用的，现在看上去并不是几个数量级的差，或许现在的标准库已经作了优化吧。

