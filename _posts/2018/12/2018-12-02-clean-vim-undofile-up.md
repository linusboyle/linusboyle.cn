---
title: 清理Vim的历史记录文件(undofile)
date: 2018-12-02
tags:
- vim
- bash
- scripting
- 脚本
layout: post
excerpt: 简单快捷地清理vim编辑文件储存的undo文件
---

vim提供保存修改记录的机制，它可以把文件的修改历史记录在单独的undofile中。这样下一次启动vim进行编辑的时候，不会丢失缓冲区的编辑历史。

说的具体一点，我的配置如下:

```vim
"undo tree
let s:undo_dir = expand("~/.cache/undodir")

if !isdirectory(s:undo_dir)
    silent! call mkdir(s:undo_dir, 'p')
endif

if has("persistent_undo")
    set undodir=~/.cache/undodir
    set undofile
endif
```

所有的记录被统一储存在一个目录下，避免污染工作区（backup同理）

当使用一段时间后，undofile将占据一定的空间。这其中绝大部分是没有必要的，因为

1. 也许原来的文件早已被删除
2. 文件已经很久没有被编辑过
3. ...

如果要按照1.来解决这个问题会比较棘手，因为vim的undofile命名方式是这样的：

> %home%linusboyle%dev%program%dsa%pa3%max%.ycm_extra_conf.py

并且，其中一部分文件是在临时挂载的文件系统中，要分辨源文件是否存在很困难。

因此，我最终选择了用编辑时间来判定是否保存历史记录，bash script实现，脚本如下:

```shell
#!/bin/bash

# cleans unused vim undofile 
# This script assumes that you store undofile in a seperated directory

UNDO_PATH="$HOME/.cache/undodir"
THRESHOLD="30" # in day, if the last modified time is beyond this value, the undofile will be deleted
REMOVE_LOG="/tmp/undoclean-log"

E_WRONGARGS=65
printhelp() {
    echo "Usage: `basename $0` [THRESHOLD] [PATH]"
    exit $E_WRONGARGS
}

case "$1" in
    "" ) ;;
    *[!0-9]* ) printhelp ;; 
    * ) THRESHOLD=$1 ;;
esac

if [ -n "$2" ]
then
    if [ -d "$2"]
    then
        UNDO_PATH=$2
    else
        echo "$2 is not a directory!"
    fi
fi

echo "This script will deleted all the vim undo file in $UNDO_PATH that has not been modified for at least $THRESHOLD days"

find "$UNDO_PATH" -mtime "+$THRESHOLD" -print -delete | tee "$REMOVE_LOG"

echo "done, a log has been created at $REMOVE_LOG"
```

感谢[Shlomi Fish](https://www.shlomifish.org/) 和 Lifepillar 在vim邮件列表的指导，我才能想到用修改时间来界定。
