---
date: 2018-07-22
layout: post
title: 在VIM中正确加载bash别名
tags: 
- bash
- linux
- Vim
- configuration
excerpt: 关于如何在vim中使用bash定义的别名
---

在vim里调用外部命令时使用的sh虽然不一定是bash，不过大多数情况下是的。这篇文章讨论linux的情况，在这里sh是bash的软连接。

如果要严格设置sh的类型，加上这么一句就行了：

```vim
let g:is_bash	   = 1
```

回到别名的问题，bash在不是交互模式的情况下是不会读取.bashrc的，所以所有的用户设置自然没效果,当然，你可以设置bash -i选项来让它成为交互模式，不过想想看在vim里运行一个命令之后持续的交互模式——我想我们还是想个别的法子吧。

比如我把grep映射到grep -i，在vim里执行内建:grep! hello 不会匹配Hello。你可以手动指定-i参数，不过这显然不是一劳永逸的方法。

根据[这里](https://stackoverflow.com/questions/4642822/commands-executed-from-vim-are-not-recognizing-bash-command-aliases)的解决方法，最好的办法是在`.vimrc`里加入：

```vim
let $BASH_ENV = "~/.bash_aliases"
```

在这里我也建议把别名单独放在一个文件里，因为在诸如此类的情况下整个`.bashrc`是没有意义的。何况`.bashrc`里有一些启动时输出的命令，这可不是我们在vim里想看到的

还有最后一件事，记得在`.bash_aliases`开头加上这么一句：

```sh
shopt -s expand_aliases 
```

bash非交互模式似乎不会设置`expand_aliases`选项，为保险手动设置一下
