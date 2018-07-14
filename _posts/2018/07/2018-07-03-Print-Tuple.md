---
title: 遍历std::tuple
layout: post
tag: 
- c++ 
- template 
- tuple
date: 2018-07-03
excerpt: 本文讨论了泛型编程中遍历tuple元素，应用自定义函数对象的方法。实现了针对tuple的for_each函数
---
在c++中遍历元组的基本方法,最终实现传入任意函数对象作用于元组的元素

### 打印元组
这里用cout举例，提供一种方法遍历元组的元素并依次输出，显然这需要可变参数和递归。首先定义一个辅助函数确定数组大小：(这里参考了[Rajasekharan Vengalil的博客](https://blogorama.nerdworks.in/iteratingoverastdtuple/) )

```cpp
 template<typename... Ts>
 void print(std::tuple<Ts...>&& t) {
     constexpr auto size = std::tuple_size<std::tuple<Ts...>>::value;
     print_tuple<size - 1, Ts...>{}(std::forward<std::tuple<Ts...>>(t));
 }
```

print_tuple是一个结构体，不实现为函数的原因很简单：函数模板不能偏特化，如果用函数重载的方法来实现，由于参数不是常量表达式而无法编译。实现的方法如下：(注：可以用模板函数实现，但那可能需要index_sequence，这里的实现只需要c++11)

```cpp
template<int index, typename... Ts>
 struct print_tuple {
     void operator() (std::tuple<Ts...>&& t) {
         print_tuple<index - 1, Ts...>{}(std::forward<std::tuple<Ts...>>(t));//1
         std::cout << std::get<index>(t) << " ";//2
     }
 };

 template<typename... Ts>
 struct print_tuple<0, Ts...> {
     void operator() (std::tuple<Ts...>&& t) {
         std::cout << std::get<0>(t) << " ";
     }
 };
```

一个小测试：

```cpp
int main(){
    print(std::make_tuple(2,2,"fal",true,false,"hah"));
}
```

输出：

```shell
~/dev/program/template$./print 
2 2 fal 1 0 hah
```
(忘了std::boolnalpha，anyway...)

如果要倒序输出只需要把1和2调换就行了

### 过滤单一类型的数组元素

用模板递归检测元素类型，如果是需要过滤的类型则删去，最后返回过滤后的元组。

这里用到std::tuple_cat 用途顾名思义，用来连接元组

```cpp
template< typename T>
struct Filter
{
    static constexpr auto func()
    {
        return std::tuple<>();//递归终止，返回空元组
    }

    template< class... Args >
    static constexpr auto func(T&&, Args&&...args)
    {
        return Filter::func(std::forward<Args>(args)...);//如果是要过滤的类型则直接跳过
    }

    template< class X, class... Args >
    static constexpr auto func(X&& x, Args&&...args)
    {
        return std::tuple_cat(std::make_tuple(std::forward<X>(x)), Filter::func(std::forward<Args>(args)...));
    }
};
```

通过上面的print函数来测试一下：
```cpp
int main(){
    std::cout<<std::boolalpha;

    auto result=Filter<bool>::func(1,1,5,true,false,1,"really");

    print(std::make_tuple(1,1,5,true,false,1,"really"));
    std::cout<<std::endl;

    print(std::move(result));
    std::cout<<std::endl;

    return 0;
}
```

输出：
```
~/dev/program/template$./filter
1 1 5 true false 1 really 
1 1 5 1 really 
```
所有布尔值都被过滤掉了

### 能否抽象出迭代器，传入函数对象？

这个问题我想了想，结合一些资料，认为是可以实现的。观察print的实现方法，其实和cout没有实际的耦合关系，所以可以把函数对象作为参数传进去。沿用print_tuple的结构，增加一个模板参数即可，实现函数for_each如下：

```cpp
template<int index,typename F,typename... Ts>
struct apply_tuple {
    void operator() (F func,std::tuple<Ts...>&& t) {
        apply_tuple<index - 1,F,Ts...>{}(func,std::forward<std::tuple<Ts...>>(t));
        std::apply(func,std::make_tuple(std::get<index>(t)));
    }
};

template<typename F,typename... Ts>
struct apply_tuple<0,F,Ts...> {
    void operator() (F func,std::tuple<Ts...>&& t) {
        std::apply(func,std::make_tuple(std::get<0>(t)));
    }
};

template<typename... Ts,typename Func>
void for_each(std::tuple<Ts...>&& t,Func F) {
    const auto size = std::tuple_size<std::tuple<Ts...>>::value;
    apply_tuple<size - 1,Func,Ts...>{}(F,std::forward<std::tuple<Ts...>>(t));
}
```

传入任意函数对象，比如lambda表达式：
```cpp
int main(){
    auto test=std::make_tuple(1,1,"2",true,false,290,"hihi");
    for_each(std::move(test),[](const auto& x){std::cout<<x<<std::endl;});
}
```
这个实现使用了c++17标准的std::apply,但它可以被c++11简单地实现。当然这个实现还有简化的地方(老实说，不怎么优雅)，比如可以直接用index_sequence展开，而不是递归。

c++的模板很奇妙，接下来有时间再深入了解。
