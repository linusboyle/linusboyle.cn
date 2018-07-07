---
title: C++实现通用函数返回值记忆（Memoization）"装饰器"
date: 2018-07-07
tags: c++ memoization template functional
layout: post
---

Memoization的翻译不是很确定，姑且叫它“返回值记忆”吧，意思是记录下函数输入和对应的返回值，以更多的空间占用换取时间效率。这种技巧在很多算法里都有应用，比如你要实现一个计算斐波那契数列的函数，最简单的就是通过递归。不过这种方式相当低效，同一个值被重复计算了多次。通常的做法是开一个数组存下计算出的值，如果之前已经算过这个值，就直接返回，而不用再递归多次。

Python有一种装饰器，可以把函数转化为有返回值记忆功能的函数。

```python
def memoize(fn):
     cache = {}
     def memoized_fn(*args):
         if args not in cache:
             cache[args] = fn(*args)
         return cache[args]
     return memoized_fn
```
我对python不是很了解，不过其装饰器可以对递归函数使用（原理未知……），这里笔者尝试在C++中实现类似的功能,实现对任意纯函数（pure function）的记忆化，首先先模仿装饰器写一个模板试试，这是比较直白简单的：

```cpp
#include <functional>
#include <map>
#include <boost/timer.hpp>
#include <iostream>
template <typename ReturnType, typename... Args>
std::function<ReturnType (Args...)> memoize(std::function<ReturnType (Args...)> func)
{
    std::map<std::tuple<Args...>, ReturnType> cache;
    return ([=](Args... args) mutable  {
            std::tuple<Args...> t(args...);
            auto result=cache.lower_bound(t);
            if (result == cache.end()||result->first!=t){
                result = cache.insert(result,std::make_pair(t,func(args...)));
            }
            return result->second;
    });
}

int main(){
    //use
    //模板不会隐式转换类型，所以得显式构造std::function
    auto memoizedfun=memoize(std::function<int(int)>(
        [](int x){
            for(int i=0;i<100000000;i++){
                //sleep...
                ;
            }
            return x;
        })
    );
    boost::timer timer;
    std::cout<<memoizedfun(23333)<<std::endl;
    std::cout<<timer.elapsed()<<std::endl;

    timer.restart();

    std::cout<<memoizedfun(23333)<<std::endl;
    std::cout<<timer.elapsed()<<std::endl;
}
```

输出显而易见，第二次调用要比第一次快得多：
```
23333
0.164355
23333
3e-06
```
这样做的好处显而易见：当函数对象生命周期结束，缓存也会被释放。

这个简单的模板对于没有使用递归的函数来说已经够用了。实际上即使是对于递归的函数，只要注意一下函数体内的写法，也能正确地被“装饰”，举个斐波那契数列的例子：

```cpp
std::function<int(int)>f;

int fib(int n){
    if(n<2){
        return n;//0th->0,1st->1
    }
    return f(n-1)+f(n-2);
}

int main(){
    f=memoize(std::function<int(int)>(fib));

    boost::timer timer;
    std::cout<<f(40)<<std::endl;
    std::cout<<timer.elapsed()<<std::endl;

    return 0;
}
```

*不过这种写法还不如手动开数组呢……*

要找到和python一样可以作用于任意函数的装饰器是很困难的，至少对于递归函数，其内部实际自调用情况是不能被高阶函数所知的，所以在上面的代码中fib的递归用的是记忆化后的版本f。而且看上去似乎不怎么**优雅**。

我们需要对递归函数换一种思路。可以借鉴λ演算里Y-不动点组合子(Y-combinator)的概念，将要递归的函数作为一个参数传入高阶函数（lambda）中，再对此高阶函数进行返回值记忆的处理。这需要c++14 泛型λ表达式的支持

```cpp
struct holder {};

template<class Sig, class F>
struct memoizer;

template<class R, class...Args, class F>
struct memoizer<R(Args...), F> {
    private:
        F base;
        std::map<std::tuple<Args...>, R> cache;
    public:
        template<class... Ts>
        R operator()(Ts&&... ts)
        {
            auto args = std::tie(ts...);
            auto it = cache.find( args );
            if (it != cache.end())
                return it->second;

            auto&& retval = base(*this, std::forward<Ts>(ts)...);
            cache.emplace( std::move(args), retval );

            return decltype(retval)(retval);
        }

        memoizer(memoizer const&)=default;
        memoizer(memoizer&&)=default;
        memoizer& operator=(memoizer const&)=default;
        memoizer& operator=(memoizer&&)=default;
        memoizer() = delete;
        template<typename L>
        memoizer( holder, L&& f ):base( std::forward<L>(f)){}
};

template<class Sig, class F>
memoizer<Sig,F> memoize( F&& f ) { return {holder{}, std::forward<F>(f)};}

int main(){
    auto fib = memoize<size_t(size_t)>(
        [](auto&& fib, size_t i)->size_t{
            if (i<=1) return i;
                return fib(i-1)+fib(i-2);
        }
    );
    std::cout<<fib(100)<<std::endl;
}
```
读者可以对比一下原来O(2^n)级的递归算法的函数和这个新生成的函数对象的效率(注意表达式的返回值可能需要显式的指定，gcc7.2推断出的返回值是错误的)。本文到此告一段落，后续还可以再扩展，比如用一个枚举表示是否递归，然后写一个辅助函数决定使用哪种实现。
