---
title: 一维线段树的2n空间实现
date: 2019-01-03
layout: post
tags:
- 算法
- 线段树
excerpt: 新年第一篇，简要介绍一下线段树的通用O(2n)空间实现
---
> 2019年到了，祝愿大家心想事成，万事如意。 :)

网上很多人说线段树需要O(2n) ~ O(4n) 的空间。对于线段树这样一个完全二叉树来说，怎么可能没有严格2n的实现方式呢？

原因在于对于线段树结构的理解不同。线段树自底向上构建，的确是一棵完全二叉树；然而大部分递归构建的线段树，本身采用的是二分策略，必会导致大量的无用空间。这里简要说明一下自底向上、迭代地构建线段树的方法:

假设输入长度为size，开辟2*size的空间足矣。构建步骤如下：
- 将原数据复制到开辟的[size, 2*size - 1]区间内
- j从size-1到1迭代，每一步将秩为2j和2j+1的数据**合并**，储存在秩为j处。

当更新原秩为index处的数据 操作如下：
- 先直接更新size+index位置处的数据
- 令i从(size+index)/2开始迭代，每一步合并秩为2i和2i+1处的数据，赋值于秩为i处。每一步i/=2
- 当i小于等于1 结束

查询[left, right)区间的数据之*“和”* 通过递归实现如下：
- 若left是偶数，则合并left处数据，和递归[left+1,right)后的结果
- 若right是偶数，则合并right-1处数据，和递归[left,right-1)后的结果
- 否则，返回递归[left/2,right/2)的结果
- left >= right 时停止(返回空)

**合并**是关键的一步，应该由使用者自己给定。因此设计泛化的线段树如下:(其中，具体怎么合并两个数据项由函数对象_merge指定，比如**求和**操作可以用lambda实现为：`[](auto a, auto b){return a+b}`)

```cpp
template <typename T, typename F>
struct SegmentTree {
private:
    T query_p(int left, int right) {
        while (left >= right) {
            return T();
        }

        if (left & 0x1) {
            return _merge(_data[left], query_p(left + 1, right));
        } else if (right & 0x1) {
            return _merge(query_p(left, right - 1), _data[right - 1]);
        } else {
            return query_p(left >> 1, right >> 1);
        }
    }

    const F _merge;
    T* _data;
    const int _size;
public:
    using data_type = T;

    SegmentTree(int size, T* data, F merge): _size(size), _merge(merge) {
        _data = new T[_size << 1];

        // copy and construct leaves
        for (int i = 0; i < _size; ++i) {
            _data[i + _size] = data[i];
        }
        build();
    }

    ~SegmentTree() {
        if (_data)
            delete[] _data;
    }

    void build() {
        for (int i = _size - 1; i > 0; --i) {
            _data[i] = _merge(_data[i << 1], _data[(i << 1) + 1]);
        }
    }

    void update(int index, T value) {
        int i = index + _size;
        _data[i] = value;

        while (i > 1) {
            i >>= 1;
            _data[i] = _merge(_data[i << 1], _data[(i << 1) + 1]);
        }
    }

    T query(int left, int right) { // query the range [left, right)
        left += _size;
        right += _size;

        return query_p(left, right);
    }
};
```

空间复杂度为严格$\Omega(2n)$，实际运用时需要加上运行时边界检查。
