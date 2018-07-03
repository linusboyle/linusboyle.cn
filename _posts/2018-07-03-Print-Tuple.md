---
title: 遍历std::tuple
layout: post
tag: c++ template tuple
date: 2018-07-03
---
在c++中遍历元组的基本方法

### 打印元组
这里用cout举例，提供一种方法遍历元组的元素并依次输出，显然这需要可变参数和递归。首先定义一个辅助函数确定数组大小：(这里参考了[Rajasekharan Vengalil的博客](https://blogorama.nerdworks.in/iteratingoverastdtuple/) )

```cpp
 template<typename... Ts>
 void print(std::tuple<Ts...>&& t) {
     constexpr auto size = std::tuple_size<std::tuple<Ts...>>::value;
     print_tuple<size - 1, Ts...>{}(std::forward<std::tuple<Ts...>>(t));
 }
```

print_tuple是一个结构体，不实现为函数的原因很简单：函数模板不能偏特化，如果用函数重载的方法来实现，由于参数不是常量表达式而无法编译。实现的方法如下：

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
(忘了std::booleanalpha，anyway...)

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
