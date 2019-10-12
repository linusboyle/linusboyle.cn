---
title: 记无冬之夜1 Linux原版
date: 2019-08-29
layout: post
tags: 
- 游戏
- crpg
- 龙与地下城
- 无冬之夜
- linux
excerpt: 本文记录了我运行原生Linux版无冬之夜1的坑和解决办法
---

> 注意：本文并不系统，只是随手写一点运行遇到的问题和解决方法。

无冬之夜是比较老的游戏了，前几年Beamdog出了加强版，和BG，PST，IWD一样有原生Linux支持，这几个DND游戏我都是在Linux上玩的。无冬之夜之前在Win上玩过一段时间，实在不习惯3版规则，中途给弃了，出了加强版也没去玩。

偶然的机会找到了无冬之夜1原版的Linux客户端。版本号是1.6.9，可以用社区的补丁升到1.7.1。解压缩之后，里面还带了当年的Linux版服务器。感觉Linux版客户端只是服务器附带的而已。

把该解压的归档全部解压之后大概是这样的：

![nwn-1](/assets/images/2019/nwn-1.png)

我的发行版是OpenSuSE TumbleWeed 64bit，5.1的内核，运行的时候缺了一些32位的包。直接用zypper装上就可以，作为一个古董游戏，幸好没出unresolved symbols这种错。另外，游戏的启动脚本里用了自带的老版本libSDL1.2，建议修改启动脚本，用系统提供的较新的SDL1.2会好一些。

这是我的启动脚本：
```sh
#!/bin/sh
cd "`dirname "$0"`"

# This script runs Neverwinter Nights from the current directory
export SDL_MOUSE_RELATIVE=0
export SDL_VIDEO_X11_DGAMOUSE=0

# If you do not wish to use the SDL library included in the package, remove
# ./lib from LD_LIBRARY_PATH
export LD_LIBRARY_PATH=./miles:$LD_LIBRARY_PATH

#NWMovies
#export LD_PRELOAD=./nwmovies/nwmovies.so

./nwmain $@
```

这里，我把无冬的过场动画全都禁用掉了。无冬的Linux版本来是没有动画支持的，nwmovies补丁是nwnlinux社区做的解决方案，现在代码在[Github](https://github.com/nwnlinux/nwmovies) 上。这个工具已经很久没人维护了，里面的动态链接库还是在Ubuntu 8.04上编译的，我的机子加载不了。遗留的C代码用的还是很老的Linux API，要自己编译的话还得去改源代码，索性弃之。

BinkPlayer播放器也有不少兼容性问题。不仅会独占声卡，还会把我的X11分辨率给改了。我加了一大堆软连接以后，总算是可以把它跑起来，手动播放出动画。就是动画结束的时候SDL内部会爆段错误，然后把程序卡住。应该是内核的问题，所以放弃之。

解决（禁用）动画之后就是声音的问题。进入游戏后，不管是音效还是背景音乐都无法播放，在声音选项中，找不到我的声音设备。网上游弋了一圈之后，大概知道这应该是ALSA和PulseAudio的问题。无冬之夜是需要直接访问声卡的，不能用PA，而且当年游戏发售的时候，PA还没正式Release呢。PulseAudio有ALSA兼容层，但显然没办法处理好这种情况。

解决方法是用pasuspender工具临时暂停PA服务端，让无冬直接独占声卡就行了。

```sh
pasuspender -- ./nwn
```

第一次运行依旧没有声音，退出去再运行就行了。

到这儿大问题就解决完了，创建了一个角色试了试，还没出新手地牢，到目前为止运行非常良好。

补充：已经差不多打完第二章，几乎没有bug，可以说是很完美了。打了一个快速休息的mod，也可以正常工作。之后再试试装prc和自定义模组。
