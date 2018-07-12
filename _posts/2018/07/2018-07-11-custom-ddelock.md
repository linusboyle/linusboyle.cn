---
title: Scrot+ImageMagick增强dde-lock
date: 2018-07-11
tags: deepin scrot linux
layout: post
excerpt: 增强（或者说美化）deepin桌面环境的锁屏界面
---

之前一段时间一直在用i3，不过网卡出了点问题，最近用回dde（主要是懒得注销了）。以前在i3定制的很多东西不太方便，比如动态模糊锁屏。

其实这也不是什么难事，之前的lock脚本一样可以用，只不过和i3lock的密钥验证环相比还是dde的lock界面好一些。问题主要就是每一次启动的时候截图+模糊。

注意本文写作使用的是deepin15.6，之后的存放位置可能会改。

首先，dde-lock使用的图片和dde-laucher的背景图片一样，都是存放在**/var/cache/image_blur/**里，没错，里面有十多张预置的图片，我建议不要修改这些图片，自己先随便选一张图片，在桌面右键把它设置为锁屏图片。

这个时候在上面说的文件夹用ls -l之类的命令找到最新添加的图片，记下它的名字，删除之。然后添加符号连接（这一步是为了避免在var目录直接操作导致的权限问题，毕竟谁也不想在锁屏的时候输密码）到/tmp里的随便一个目录。

然后只需要一个小脚本就搞定了，我的如下（注意更改符号链接的目标）：
```bash
#! /bin/bash
# linusboyle 2018-07-11
# blur the screen and lock

tmpFile="/tmp/dlock_img.png"#你的路径
scrot -m "$tmpFile" #不用深度截图是因为慢……而且有bug
convert "$tmpFile" -blur 80x5 "$tmpFile"
dde-lock
```
convert 的参数非常多，你可以配置出各种风格，比如科幻风，jojo风之类的……不过这里只演示了高斯模糊（x号后面的是高斯模糊的参数，有兴趣的可以自己改来看看）。

最后在控制中心把快捷键绑定上。按下super+l即可。

演示：（深度录屏的gif录制失真比较严重……）
![showcase](/assets/images/2018/lockshowcase.gif)

## 继续折腾
下面的折腾可能会导致不明的问题，而且在系统升级的时候可能会被覆盖

你可能会说，我按电源键用的还是原来的dde-lock，这让人很不爽。当然可以一次性解决这个问题，如果你愿意在待机的时候多等两秒，那就继续看下去吧。

控制按下“锁屏”时运行程序路径的是dbus的服务配置文件，我们只需要简单+粗暴地把`/usr/share/dbus-1/services/com.deepin.dde.lockFront.service`里的程序改成上面的脚本就可以了。sudo vim改之，按下电源键，现在你需要在睡眠之前等上两秒……老实说，这并没有什么意思，最好也别这样折腾。
