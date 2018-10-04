---
title: 使用C++17实现邱奇数
layout: post
tag:
- c++
- functional
- template
- lambda
- church numerals
date: 2018-09-18
excerpt: 使用c++来写一个邱奇数的实现，包括加、乘、减一操作
---

邱奇数是啥？这个我并不打算多解释，网上有很多文章已经叙述过了，也带了lua，javascript，haskell等语言的实现。这里用cpp简单的实现一下，因为是静态强类型语言，我暂时还没办法实现幂和减法（两者需要一点动态语言的特性，至少我是这么理解的）

那么先从0和1开始吧，阅读之前可能需要了解一下cpp的泛型编程和函数式编程的内容

## 邱奇数

使用using关键字，我们先定义两个泛型类型；

```cpp
template<typename R>
using church_number_func_t = std::function<R(R)>;

template<typename R>
using church_number_t = std::function<church_number_func_t<R>(church_number_func_t<R>)>;
```

`chuch_number_t`即是邱奇数的类型（也许c++的面向对象在这里能发挥点作用，不过暂且用泛型写吧……），从它的原型可以看出，邱奇数是一类函数，它接受一个函数为参数，返回相同原型的另一个函数。

现在定义0:
```cpp
template<typename R>
auto church_zero = church_number_t<R>([](church_number_func_t<R> f)->church_number_func_t<R>{
    return [f](R x)->R{
        return x;
    };
});
```

0不对x做任何操作，接下来是1:

```cpp
template<typename R>
auto church_one = church_number_t<R>([](church_number_func_t<R> f)->church_number_func_t<R>{
    return [f](R x)->R{
        return f(x);
    };
});
```

1对x应用f一次，对于任意的邱奇数m，都应该有

$$
m(f)(x) = f^m(x)
$$

第一个m是邱奇数类型，第二个m是自然数标识的邱奇数，为了方便，下文直接用自然数表示了。

## 运算

现在通过0和1两个邱奇数，实现生成其他自然数的高阶函数，先从加法开始吧。

### 加法

```cpp

template<typename R>
constexpr church_number_t<R> church_add(church_number_t<R> first,church_number_t<R> second){
    return [first,second](church_number_func_t<R> f)->church_number_func_t<R>{
        return [f,first,second](R x)->R{
            return first(f)(second(f)(x)); 
        };
    };
}
```

邱奇加法运算将两个邱奇数顺序应用到传入的函数及参数上，简单理解为

$$
m(f)(n(f)(x)) = f^m(f^n(x)) = f^{m+n}(x) = (m+n)(f)(x)
$$

### 乘法

```cpp
template<typename R>
constexpr church_number_t<R> church_mult(church_number_t<R> first,church_number_t<R> second){
    return [first,second](church_number_func_t<R> f)->church_number_func_t<R>{
        return [f,first,second](R x)->R{
            return first(second(f))(x); 
        };
    };
}
```

邱奇乘法先将两个邱奇数顺序应用到函数，之后再以传入的参数调用：

$$
m(n(f))(x) = (n(f))^m(x) = (f^n)^m(x) = f^{mn}(x) = (mn)(f)(x)
$$

### 自减

我并不能实现幂和减法，不过简单的自减（即返回前一个自然数，0的前驱为0）是很简单就可以实现的：

```cpp
template<typename R>
constexpr church_number_t<R> church_pred(church_number_t<R> num){
    return [num](church_number_func_t<R> f)->church_number_func_t<R>{
        return [num,f](R x)->R{
            bool indicator = true;

            auto pred_func = [&indicator,f](R x)->R{
                if (indicator){
                    indicator = false;
                    return x;
                } 
                return f(x);
            };

            auto retval = num(pred_func)(x);
            return retval;
        };
    };
}
```

这里的思路是用一个布尔值标识第一次函数调用，通过c++ lambda函数捕获应用的能力，在第一次函数调用时略过，然后置布尔值为否。

#### 如何实现减法和幂

有些读者可能比较好奇减法的问题，在写出自减之后，只需要顺序执行自减x次即可。被cpp的类型所扰，要将一个邱奇数应用在另一个邱奇数上，理论上需要第二层的泛型，也就是“模板的模板”，所以到目前为止我并不能实现减法

幂也是一样，需要将邱奇数应用在邱奇数上，这是我目前很头痛的问题。

## 测试

简单的函数测试：

```cpp
#include <iostream>

int foo(int x){
    std::cout<<"hello"<<x<<std::endl; 
    return x+1;
}
```

要使用上述定义的邱奇数，首先将需要的基础邱奇数实例化；

```cpp
auto One = church_one<int>;
```

假设我们需要7，我们可以这样：
```cpp
auto Two = church_add(One,One);
auto Four = church_add(Two,Two);
auto Eight = church_mult(Two,Four);
auto Seven = church_pred(Eight);
```

ok,现在调用函数：
```cpp
Seven(foo)(1);
```

Output：
>hello1
>hello2
>hello3
>hello4
>hello5
>hello6
>hello7

现在你可以用它做一些复杂的事，不过从效率上来说，可能不怎么如意

最后我们包装一下成果，一个模板库诞生了（误

```cpp
/**
 * church_numerals.h
 * Copyright (c) 2018 Linus Boyle <linusboyle@gmail.com>
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */

#ifndef CHURCH_NUMERALS_H
#define CHURCH_NUMERALS_H

#include <functional>

namespace Church{

    template<typename R>
    using church_number_func_t = std::function<R(R)>;

    template<typename R>
    using church_number_t = std::function<church_number_func_t<R>(church_number_func_t<R>)>;

    template<typename R>
    auto church_one = church_number_t<R>([](church_number_func_t<R> f)->church_number_func_t<R>{
        return [f](R x)->R{
            return f(x);
        };
    });

    template<typename R>
    auto church_zero = church_number_t<R>([](church_number_func_t<R> f)->church_number_func_t<R>{
        return [f](R x)->R{
            return x;
        };
    });

    template<typename R>
    constexpr church_number_t<R> church_add(church_number_t<R> first,church_number_t<R> second){
        return [first,second](church_number_func_t<R> f)->church_number_func_t<R>{
            return [f,first,second](R x)->R{
                return first(f)(second(f)(x)); 
            };
        };
    }

    template<typename R>
    constexpr church_number_t<R> church_mult(church_number_t<R> first,church_number_t<R> second){
        return [first,second](church_number_func_t<R> f)->church_number_func_t<R>{
            return [f,first,second](R x)->R{
                return first(second(f))(x); 
            };
        };
    }

    template<typename R>
    constexpr church_number_t<R> church_pred(church_number_t<R> num){
        return [num](church_number_func_t<R> f)->church_number_func_t<R>{
            return [num,f](R x)->R{
                bool indicator = true;

                auto pred_func = [&indicator,f](R x)->R{
                    if (indicator){
                        indicator = false;
                        return x;
                    } 
                    return f(x);
                };

                auto retval = num(pred_func)(x);
                return retval;
            };
        };
    }

}//namespace Church
#endif
```
