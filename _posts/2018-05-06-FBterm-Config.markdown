---
layout: post
title: Fbterm相关配置
date: 2018-05-06 
tag: framebuffer linux
---

tty下终端模拟器fbterm使用framebuffer工作，所以像是fbi，fbgs之类的工具是没法使用的。我使用fbterm的目的主要有两个，一个是中文支持，一个是vim插件的支持。不过目前其开发已经停止，在本机上的效果并不完美



[根据ArchWiki](https://wiki.archlinux.org/index.php/Fbterm)，需要先进行用户组的设置，这里跳过不谈。

### 设置字体

运行一次fbterm生成`~/.fbtermrc`，在其中找到`font-names=` 和`font-size=` 两个选项，可以设置字体族和字体大小

### 配置输入法

支持fbterm的输入法有yong，fcitx，ucimf等，比较好用的是yong

fcitx应该也可以使用，但是用apt安装对fbterm的支持会卸载图形界面的fcitx，目前还不知道如何解决

在fbtermrc里找到`input-method=` 填入输入法模块，在fbterm中按下`Ctrl+Space` 调用输入法。这个快捷键应该是不能更改，所以会影响输入法本身的一些功能。

### 中文显示问题

虽然fbterm能正确识别中文，其显示依旧是？？？（可能是字体的原因）。不过在vim中使用输入法输入中文，显示都没有问题，这也是安装的最主要原因。

### Tmux

在tmux中无法运行fbterm，而如果先运行fbterm再tmux，则tmux的页面在不支持中文的同时对256色的支持也会失去，这可能和bashrc的配置有关。
