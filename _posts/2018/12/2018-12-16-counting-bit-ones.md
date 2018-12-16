---
title: 统计二进制表示中'1'的数量
date: 2018-12-16
layout: post
tags:
- 算法
- 位运算
excerpt: 描述了三种不同复杂度的算法来解决这一问题，虽然并不清楚这一应用的实际意义
---

对于一个固定长度的整数，如果要统计其二进制表示中1的数目，应该如何实现？

最简明的方法如下：
```cpp
int count(int n) {
    int retval = 0;
    int mask = 0x1;
    for (int i = 0; i < 32; ++i, mask <<= 1) {
        if (n & mask) {
            retval++;
        }
    }

    return retval;
}
```
迭代次数为二进制表示的长度，对于整数而言即$O(logn)$

借助于位运算的技巧，可以改进上述算法，使得每一次迭代去除最后一位的‘1’，从而使得迭代次数降低为实际1的数目:
```cpp
int count2(int n) {
    int retval = 0;

    while (n) {
        retval++;
        n &= (n-1);
    }

    return retval;
}
```
更加高效的一种算法是在二进制位的层面上进行**二分**，通过分组，使得1的数目逐次相加。

首先，我们需要这样一种mask：它以固定的步长分割0和1的部分，比如，mask(0)在32位整数上应该是：

> 01010101010101010101010101010101

而mask(1)则是：

> 00110011001100110011001100110011

这种二进制数可以通过除法来生成，考虑$2^n$长度的‘1’串作为除数，-1作为被除数（即二进制位全为1），则二者相除可以生成这样的串。

```cpp
inline int pow(int c) {
    return 0x1 << c;
}

inline int mask(int c) {
    return static_cast<unsigned int>(-1)/(pow(pow(c))+1);
}
```
如此，我们可以利用mask将一个数分为两组，分别对应mask的0和1，然后将这两部分的1数目相加：

```cpp
inline int round(int n, int c) {
    return (n & mask(c)) + ((n >> pow(c)) & mask(c));
}
```

由于我们注意到，当我们以$2^0$即1为组长时，1的数目就是当前比特位的值，因此，我们将其作为基，连续地通过执行代数运算，来统计1的数目

```cpp
inline int count1(int n) {
    n = round(n, 0);
    n = round(n, 1);
    n = round(n, 2);
    n = round(n, 3);
    n = round(n, 4);

    return n;
}
```
这里每一次调用round(n,c)，都假设以$2^c$为组长的部分的1的数目已经被存在相应的组内，然后在内部将相邻的组的1的数目相加。该算法巧妙地用加法实现了类似递归的过程。此方法的复杂度为$O(loglogn)$

上述方法都可以被很轻易地拓展到更大的整数，实际上也可以用于任何对象之上。
