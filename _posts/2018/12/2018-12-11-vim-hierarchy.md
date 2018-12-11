---
title: Vim自定义配置(三)：拆解vimrc
date: 2018-12-11
layout: post
tags:
- Vim
- scripting
excerpt: 作为vim进阶的必经之路，学会逐步放弃对插件的依赖和数千行的配置文件，能更好的帮助你打造自己独特的vim
---

和emacs用户不一样，用vim的人，大部分人都会装上一大堆插件，然后在vimrc里做相应的配置。有一些经验的人可能会用注释以及marker、fold等，让他们的配置文件尽量整洁。基本上每一个刚开始接触vim的人，都是从抄别人的vimrc开始的。

不过实际上vim的自定义性要发挥地淋漓尽致，需要使用者了解vim运行环境的目录结构。这篇文章首先解决最基础的问题：把长长的vimrc配置文件分解为插件。

## plugin
vim的plugin，字面意思是“插件”，不过实际上指的就是运行时自动加载的vimscript文件而已。真正意义上，或者说大家熟知的vim“插件”，对应的应该是vim原生的“pack”。究其原因在于，现在大多数vim插件并不是单一的脚本文件实现的，其本身带有自动加载的脚本、帮助文档、python或者其他编程语言的库等等。这种插件本身就是vim运行环境的一个缩影。因此vim8加入了pack的概念，引入原生的包管理机制。

vim读取vimscript脚本的路径由`runtimepath`指定，其中的每一个目录都被视为一个vim目录树的根。vim自带的`runtime`命令用于在运行时从中读取某组脚本：

```vim
runtime foo.vim
```
这会在runtimepath指定的每一个目录下寻找`foo.vim`，并且读取它。因此，如果你将一部分vim全局配置分离出去，并且用`source`命令读取，更好的方法是把脚本放在`.vim/`目录里，然后使用runtime。

默认地，启动vim时，会自动执行以下命令：

```vim
runtime! plugin/**/*.vim
```

由此我们可以迈出我们的第一步：如果你的vimrc里散布着各种各样的function、command、autocmd，可以将他们放在单独的vimscript脚本里，置于`.vim/plugin/`下的任意位置，vim会自动读取之。

## ftplugin
有一部分人热衷于通过自动命令来设置不同语言的不同参数。实际上这完全可以通过vim的ftplugin，即文件类型插件轻易的实现。ftplugin是这样一个脚本：它的名称类似于xxx.vim，xxx即是文件类型的名字，比如javascript。在vim的buffer切换为对应的文件类型时，会自动加载此脚本的内容。因此使用文件类型插件能更高效地设置不用语言的特殊参数，比如tab大小，注释字符等等。其存放位置是`.vim/ftplugin/`

需要注意在其中应该使用`setlocal`以替代`set`，并且设置`b:undo_ftplugin`，以告知vim如何恢复这些设置（在某些情况下，buffer的类型可能会发生变化）。详情可以看:h undo_ftplugin

## indent
与ftplugin类似，存放于`.vim/indent/`中的缩进配置文件定义了特定语言的缩进相关的特殊设置。你当然可以把它们和ftplugin放在一起，不过我更喜欢遵守vim本身的规则。

## ftdetect
ftdetect内的脚本用于提供额外的文件类型检测。比如perl：

```vim
autocmd filetypedetect BufNewFile
    \ ?*/bin/?*
    \,?*/libexec/?*
    \,?*/scripts/?*
    \ if filereadable(expand('<afile>:p:h:h') . '/Makefile.PL')
    \|  setfiletype perl
    \|endif
```

这里根据自己的实际情况进行设置，上述自动命令告知vim，如果创建的新文件在某些特定文件夹，且同一文件层次内有`Makefile.PL`，那么设置创建的新文件为perl。编辑完成后将其保存为`.vim/ftdetect/perl.vim`即可

## vimrc

vimrc是用来**设置**参数的地方，也就是说，用于设置vim选项、定义全局变量、显式加载文件或插件、定义字符映射等。函数具体的实现，本身就不应该放在vimrc里。因为在大多数情况下，编写vimscript都是比较费时和痛苦的事情，所以将自己自定义的功能尽量做得完善和可拓展是完全有必要的。就像优秀的插件总是提供大量的可配置项一样，在自己编写vim脚本的时候，也需要充分考虑自己可能发生变化的需求。

经过这几步，基本上vimrc能缩减很多。目前我的设置带上第三方插件的映射，一共是三百行。通过这些简单的分离和重构，可以很方便日后的修改，vim用户建立起自己的vim目录，还是相当有必要的。但是这里说的远远不是vim脚本体系的全部，仅仅介绍了不涉及插件编写的部分。日后有机会，会再介绍vim自动读取、compiler等内容。
