---
layout: post
title: 记zsh的崩溃
date: 2018-05-17
tags: zsh linux
excerpt: 简要记录zsh崩溃的后果和解决方案。
---

喜欢折腾的性子总是改不了——为了在deepin上使用kali的工具，不知道怎么回事让zsh崩溃了

我只用了一个主题，其他的完全按照ohmyzsh的默认来，一开始的症状是打不开终端，等到字符界面也无法进入的时候，连带我的tty1也崩溃了

最终还是登上图形界面用商店装了konsole才发现zsh崩溃的问题 赶紧手动进入bash 切换到root后chsh

我觉得用linux还是平实一点比较好……谨记一次教训于此

补充：解决卸载zsh后vim和gdb等程序仍然使用zsh的问题：在bashrc中加入

```sh
export SHELL="/bin/bash"
```

