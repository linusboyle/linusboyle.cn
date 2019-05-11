---
title: snapd无法加载apparmor配置解决方法
date: 2019-05-11
layout: post
tags:
- linux
- apparmor
- snap
excerpt: 据称这是snap的一个老问题，因为无法自动加载apparmor配置而导致的重启后应用全部失效问题。
---

很久没有写技术文章了:)

Spotify只提供deb的安装包，Tumbleweed的源里的deb转rpm脚本因为缺少依赖运行失败，所以我只好转向snap或者appimage了。Spotify运行一切良好，体验不错，建议大家都可以试试。只不过电脑重启之后，snapd就因为apparmor的问题停止工作了，并抛出如下错误:

```
cannot change profile for the next exec call: No such file or directory 
snap-update-ns failed with code 1: No such file or directory
```

systemd日志显示出了这是spotify的apparmor配置没加载的原因。在我的发行版，snapd应用的apparmor配置在`/var/lib/snapd/apparmor/profile`目录底下，不知道为什么snapd没办法自动加载，那就只好自己来喽：

```
sudo apparmor_parse -r /var/lib/snapd/apparmor/profile/*
```

一切正常，完美。

![showcase](/assets/images/2019/spotify.png)
