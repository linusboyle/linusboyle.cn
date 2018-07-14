---
layout: post
title: 如何在终端打印图片
date: 2018-06-28
tags: 
- shell 
- linux 
excerpt: 描述了我修改的一个小脚本，可以在特定的终端支持直接打印图片。
---

## 依赖
* w3m
* bash

## 脚本
这个脚本参考了[z3bra的博客](http://blog.z3bra.org/2014/01/images-in-terminal.html)

使用w3m自带的图片查看工具，在当前终端打印图片。为了不影响后续的命令输入，默认清屏。

它会根据终端的大小调整图片大小，尽量填满终端，并在下部显示新的PS1。对终端的色彩要求没有测试

下面是脚本:

```shell
#!/bin/bash
#
# z3bra -- 2014-01-21
# linusboyle modify this a little bit
#show img in terminal using w3mimgdisplay


test -z "$1" && exit

clear

W3MIMGDISPLAY="/usr/lib/w3m/w3mimgdisplay"
FILENAME=$1
FONTH=21 # 如果没有占满终端或过大，修改此值
FONTW=11  # 此值也是一样 这两个值是终端每行每列的大小，经过计算并不是像素的个数
COLUMNS=`tput cols`
LINES=`tput lines`

read width height <<< `echo -e "5;$FILENAME" | $W3MIMGDISPLAY`

max_width=$(($FONTW * $COLUMNS))
max_height=$(($FONTH * $LINES))

if test $width -gt $max_width ; then
    height=$(($height * $max_width / $width))
    width=$max_width
fi

if test $height -gt $max_height ; then
    width=$(($width * $max_height / $height))
    height=$max_height
fi

w3m_command="0;1;0;0;$width;$height;;;;;$FILENAME\n4;\n3;"

tput cup $(($height/$FONTH)) 0
echo -e $w3m_command|$W3MIMGDISPLAY
```

截图：![pic](/assets/images/2018/screen.png)
