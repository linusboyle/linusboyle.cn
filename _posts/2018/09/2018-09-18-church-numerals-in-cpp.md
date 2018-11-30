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
excerpt: 使用c++来写一个邱奇数的实现，包括加、乘、减等算数操作，也包括与或非等逻辑操作。最后还实现了一部分控制流
---

邱奇数（或者说邱奇编码）是什么？这个我并不打算多解释，网上有很多文章已经叙述过了，也带了lua，javascript，haskell等语言的实现。不了解的同学看一看[wiki](https://en.wikipedia.org/wiki/Church_encoding)的解释就能明白，本身不是特别难懂(比起Y不动点是简单多了，lol)。这里用cpp简单的实现一下，因为是静态强类型语言，我暂时还没办法实现幂（需要一点动态语言的特性，至少我是这么理解的）

那么先从0和1开始吧，阅读之前可能需要了解一下cpp的泛型编程和函数式编程的内容。文章分为两个部分，第一部分是版本一，强类型绑定的实现，没有减法和幂，属于我强行泛型的实现，如果不感兴趣的同学请直接看第二部分代码；第二部分是纯函数式编程，主要用到c++14的泛型λ表达式和c++17的if constexpr，实现了大部分邱奇编码和减法运算，没有幂运算

## 内容大纲

* TOC
{:toc}

## 老办法：泛型

第一种办法，我们先用模板和cpp本身的类型系统，尝试用泛型结合lambda表达式来实现。这一部分只用到c++14的泛型别名

### 邱奇数

使用using关键字，我们先定义两个泛型；

```cpp
template<typename R>
using church_number_func_t = std::function<R(R)>;

template<typename R>
using church_number_t = std::function<church_number_func_t<R>(church_number_func_t<R>)>;
```

`chuch_number_t`即是邱奇数的类型，从它的原型可以看出，邱奇数是一类函数，它接受一个函数为参数，返回相同原型的另一个函数。

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

### 运算

现在通过0和1两个邱奇数，实现生成其他自然数的高阶函数，先从加法开始吧。

#### 加法

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

#### 乘法

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

#### 自减

写到这儿，我尝试了实现幂和减法，但并不能很好地实现，不过简单的自减（即返回前一个自然数，0的前驱为0）是很简单就可以实现的：

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

##### 如何实现减法和幂

有些读者可能比较好奇减法的问题，在写出自减之后，只需要顺序执行自减x次即可。被cpp的类型所扰，要将一个邱奇数应用在另一个邱奇数上，由于邱奇数本身类型不定，需要泛型，而邱奇数本身已经经过了一次抽象。所以到目前为止我并不能实现减法

幂也是一样，需要将邱奇数应用在邱奇数上，这是我用泛型实现很头痛的问题。

### 测试

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
>
>hello2
>
>hello3
>
>hello4
>
>hello5
>
>hello6
>
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

## 函数式

我们都爱函数式，lol。虽然我不怎么精通，但显然这里用它要方便得多

### 上述实现的问题

问题很多:
1. 无法处理void类型：因为我们太过于执着于用类型来限定函数的原型，我们不得不使用std::function。然而它是无法处理void(void)的。在这种情况下，只好用模板偏特化。这会让代码长度几乎变为两倍。真是血的教训
2. 无法实现减法。类型在这里是一个巨大的负担，因为我们其实不需要在这里检查类型。但是它却限定了邱奇数的类型，使得我们没办法把一个邱奇数作用于另一个。

综上，我们需要一种完全泛型，同时又保有函数式特点的工具——c++14的泛型λ函数

### 实现
这里开门见山地贴代码：

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

namespace Church {

    constexpr auto church_true = [](auto&& first) constexpr{
        return [&](auto&&) constexpr {
            return first;
        };
    };//logic true

    constexpr auto church_false = [](auto&&) constexpr{
        return [](auto&& second) constexpr {
            return second;
        };
    };//logic false

    constexpr auto church_and = [](auto&& lboolean) constexpr {
        return [&](auto&& rboolean) constexpr{
            return lboolean(rboolean)(lboolean);
        };
    };//logic and

    constexpr auto church_or = [](auto&& lboolean) constexpr {
        return [&](auto&& rboolean) constexpr{
            return lboolean(lboolean)(rboolean);
        };
    };//logic or 

    constexpr auto church_not = [](auto&& boolean) constexpr{
        return boolean(church_false)(church_true);
    };//logic not

    constexpr auto church_xor = [](auto&& lboolean) constexpr {
        return [&](auto&& rboolean) constexpr{
            return lboolean(church_not(rboolean))(rboolean);
        };
    };//logic xor

    constexpr auto church_if = [](auto&& boolean) constexpr {
        return [&](auto&& thenclause) constexpr {
            return [&](auto&& elseclause) constexpr {
                return boolean(thenclause,elseclause);
            };
        };
    };//if-then-else clause

    constexpr auto church_one = [](auto&& f) constexpr {
        return [=](auto&&... params) constexpr {
            static_assert(sizeof...(params)<=1,"parameters exceed limit");
            return f(params...);
        }; 
    };//church numeral one

    constexpr auto church_zero = [](auto&&) constexpr {
        return [](auto&&... params) constexpr {
            static_assert(sizeof...(params)<=1,"parameters exceed limit");
            if constexpr(sizeof...(params)==0){
                return;
            }
            else {
                constexpr auto retval = std::get<0>(std::make_tuple(params...));
                return retval;
            }
        };
    };//church numeral zero

    constexpr auto church_succ = [](auto&& num) constexpr {
        return [=](auto&& f) constexpr{
            return [=](auto&&... params) constexpr {
                if constexpr(sizeof...(params)!=0)
                    return f(num(f)(params...));
                else{
                    f();
                    num(f)();
                }
            };
        };
    };//church successor

    constexpr auto church_add = [](auto&& first,auto&& second) constexpr {
        return [=](auto&& f) constexpr {
            return [=](auto&&... params) constexpr {
                if constexpr(sizeof...(params)!=0)
                    return first(f)(second(f)(params...));
                else{
                    first(f)();
                    second(f)();
                }
            };
        };
    };//numeral add

    constexpr auto church_mult = [](auto&& first,auto&& second) constexpr {
        return [=](auto&& f) constexpr {
            return [=](auto&&... params) constexpr {
                return first(second(f))(params...); 
            };
        };
    };//numeral mult

    constexpr auto church_pred = [](auto&& num) constexpr {
        return [=](auto&& f) constexpr {
            return [=](auto&&... params) constexpr {
                bool indicator = true;

                auto pred_func = [&](auto&&... p) constexpr{
                    if (indicator){
                        indicator = false;
                        if constexpr(sizeof...(p)==0){
                            return;
                        } else{
                            return std::get<0>(std::make_tuple(p...));
                        }
                    } 
                    return f(p...);
                };

                auto retval = num(pred_func)(params...);
                return retval;
            };
        };
    };//numeral predecessor

    constexpr auto church_minus = [](auto&& first,auto&& second) constexpr {
        return second(church_pred)(first);
    };//numeral substraction

    //constexpr auto church_expo = [](auto&& first,auto&& second) constexpr {
        //return second(first);
    //};//numeral exponentiation,i.e first^second

    constexpr auto church_iszero = [](auto&& num) constexpr {
        constexpr auto test_func = [](auto&&) constexpr->decltype(church_false){
            return church_false;
        };

        return num(test_func)(church_true);
    };//is zero

    constexpr auto church_leq = [](auto&& first,auto&& second) constexpr {
        return church_iszero(church_minus(first,second));
    };//less than or equal

    constexpr auto church_eq = [](auto&& first,auto&& second) constexpr {
        return church_and(church_leq(first,second))(church_leq(second,first));
    };//equal

    //YOU CAN WRITE YOUR OWN IMPL OF LESSTHAN .etc LIKE THIS,SO HERE IGNORED
    
}//namespace Church

#endif
```

我想这部份比上面各种模板要好理解得多。代码本身很简单，这里也不再赘述。理解了邱奇编码的原理，或者用解释性语言已经实现过的同学一定不陌生。

最后总结一下，我个人很喜欢c++目前向Concept和constexpr的发展趋势，最终目标还是要取代模板，就像当初取代C的宏一样。没有特殊的需求，不要随便就写模板玩。多范式编程语言的优势就在于此，hhh。

全文终，感谢读者看我废话这么久。
