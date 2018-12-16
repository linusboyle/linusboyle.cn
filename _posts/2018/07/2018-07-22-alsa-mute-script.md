---
date: 2018-07-22
layout: post
title: 简易开/关静音脚本（alsa版）
tags: 
- alsa
- 脚本
- linux
excerpt: 一小段脚本，用来开关静音，同时发送通知。 
---

个人在i3wm环境写了超多脚本，比如这个静音：
```bash
#! /bin/bash
# toggle the audio output and send notification
# by linusboyle

amixer -q set Master toggle 

stat=$(amixer get Master |egrep -o '\[(on|off)\]'|head -1) 

if test $stat = "[on]" ; then
    notify-send -i audio-ready "Audio Now On"
else
    notify-send -i audio-off "Audio Now Off"
fi
```

用了简单的正则搞定状态的问题。i3的好处就是可定制性很高，把这个脚本绑定到XF86Wakeup键就行了。

当然因为用了statusbar，也可以绑到顶栏上，看一看效果：（通知服务是deepin的）

![效果](/assets/images/2018/mute.gif)
