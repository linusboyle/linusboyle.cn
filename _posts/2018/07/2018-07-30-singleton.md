---
date: 2018-07-30
layout: post
title: 基于模板和继承的单例模式
tags: 
- template
- c++
- 单例模式
excerpt: 单例模式的一小段笔记 
---

上个学期写了个python3解释器，里面抽象语法树的工厂单例是这样实现的：

```cpp
AstFactory& AstFactory::getinstance() {
    static AstFactory theinstance;
    return theinstance;
}
```

基于模板的方法比较直观，不用每个类写一遍：
（dtk的例子）

```cpp
/*!
 * a simple singleton template for std c++ 11 or later.
 *
 * example:
 *
 *  class ExampleSingleton : public QObject, public Dtk::DSingleton<ExampleSingleton>
 *  {
 *      Q_OBJECT
 *      friend class DSingleton<ExampleSingleton>;
 *  };
 *
 * Warning: for Qt, "public DSingleton<LyricService>" must be after QObject.
 */

template <class T>
class DSingleton
{
public:
    static inline T *instance()
    {
        static T  *_instance = new T;
        return _instance;
    }

protected:
    DSingleton(void) {}
    ~DSingleton(void) {}
    DSingleton(const DSingleton &) {}
    DSingleton &operator= (const DSingleton &) {}
};
```
