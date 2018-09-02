---
title: reboot()系统调用的magic number
layout: post
date: 2018-09-02
tag:
- linux
- syscall
- magic
excerpt: 挺有意思的彩蛋：）
---

好吧，查阅reboot系统调用的时候发现，头两个参数是两个magic number，而且限定死了值，否则就是EINVAL

```c
int reboot(int magic, int magic2, int cmd, void *arg);
```

manual这么写着：

> This system call  fail  (with  the  error  EINVAL)  unless  magic  equals  LINUX_REBOOT_MAGIC1  (that  is,  0xfee1dead)  and  magic2  equals LINUX_REBOOT_MAGIC2  (that  is,  672274793).   However,  since  2.1.17  also  LINUX_REBOOT_MAGIC2A (that is, 85072278) and since 2.1.97 also LINUX_REBOOT_MAGIC2B (that is, 369367448) and since 2.5.71 also LINUX_REBOOT_MAGIC2C (that  is,  537993216)  are  permitted  as  values  for magic2.  (The hexadecimal values of these constants are meaningful.)

好吧，那就转成十六进制看看：

>672274793 = 0x28121969
>85072278 = 0x05121996
>369367448 = 0x16041998
>537993216 = 0x20112000

em……是Linus Torvalds和三个女儿的生日

## 有啥用

好吧，搞清楚这些数字什么意思了，现在来想想这有什么用

其实也很简单，reboot系统调用是属于很危险的操作，如果寄存器内标识调用序号的值发生了改变，就可能启动reboot。为了预防万一，做了这层保护。除非参数的值也不小心改成这几个magicnumber（小概率事件），才会触发reboot。
