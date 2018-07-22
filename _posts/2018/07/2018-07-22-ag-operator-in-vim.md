---
date: 2018-07-22
layout: post
title: Vim自定义配置（二）：定义ag搜索操作符
tags: 
- Vim
- Vimscript
- ag
- configuration
excerpt: 通过自定义ag操作符实现快速搜索
---

vim内置的grep命令会搜索一个正则表达式，并将结果输出到quickfix。

我目前使用的插件里有leaderf，这个插件的使用方式比较像quickfix，不过支持的是文件搜索（mru，buffer等）、函数列表、tag搜索。要在源代码里跳转的话，我设置了gtags和global，通过vim的cscope支持可以实现高效跳转。不过有些时候用不到tags，或者我想使用正则表达式在全代码库里搜索的情况是用不了tags的。这个时候还是需要grep的帮忙，当然，我们把grep换成了ag。

### 版本一

首先是改一些设定（vim的quickfix只有cscope、make、grep和cfile能写入，其实挺有局限的，否则我更倾向于调用外部的ag，毕竟ag和grep的参数又不完全兼容）

```vim
" program used when running :grep
set grepprg=ag\ --vimgrep
```

接着可以按照老套路写一个vim操作符：
```vim
" grep operator
nnoremap <leader>g :set operatorfunc=GrepOperator<cr>g@
vnoremap <leader>g :<c-u>call GrepOperator(visualmode())<cr>

function! GrepOperator(type)
    let saved_unnamed_register = @@

    if a:type ==# 'v'
        normal! `<v`>y
    elseif a:type ==# 'char'
        normal! `[v`]y
    else
        "ignore multiline mode,just because it's not useful
        return
    endif

    silent! execute "grep! " . shellescape(@@)

    copen
    let @@ = saved_unnamed_register
endfunction
```

很简单的代码，应该不难懂。不过功能还是不太完善，基本上我用到ag的情况，都是要对整个代码库进行搜索，但是这只对当前目录进行了搜索，所以需要一个向上查找根目录的脚本

### 版本二

一个简单的bash脚本：
```bash
#! /bin/bash
# a small script to find root
# the root is defind as .git or .svn or .root
# by linusboyle

found="false"

ls -1a |egrep "^(.root|.git|.svn)+$" >/dev/null 2>&1 && found="true"

while [ $found == "false" ] && [ $(pwd) != "/" ]
do
    cd ..
    ls -1a |egrep "^(.root|.git|.svn)+$" >/dev/null 2>&1 && found="true"
done

if [ $found == "true" ];then
    #absolute path
    pwd
else
    echo "not found"
fi
```

为什么不用vimscript写？因为我不会……我所能想到的只有通过不断调用cd和ls和egrep，或者用find？那这样不如一次性解决，性能可能还要好些。

然后我们修改这个操作符：
```vim
" grep operator
nnoremap <leader>g :set operatorfunc=GrepOperator<cr>g@
vnoremap <leader>g :<c-u>call GrepOperator(visualmode())<cr>

function! GrepOperator(type)
    if executable('find_root')
        let saved_unnamed_register = @@
        let project_root=systemlist('find_root')[0]

        if a:type ==# 'v'
            normal! `<v`>y
        elseif a:type ==# 'char'
            normal! `[v`]y
        else
            "ignore multiline mode,just because it's not useful
            return
        endif

        if l:project_root ==# 'not found'
            silent! execute "grep! " . shellescape(@@)
        else
            silent! execute "grep! " . shellescape(@@) . " ". project_root
        endif

        copen
        let @@ = saved_unnamed_register
    else
        echo "missing find_root script,u reinstall the system again?"
        return 
    endif
endfunction
```
那句很皮的话是用来提醒我这个记性差的人的，忽略之:)

使用systemlist而不是system是因为后者会输出一个空行，既然我已经知道脚本一定会有标准输出，那么用列表的方式就是安全的，直接取数组首位得到没有空行的字符串。

看上去没什么问题，实战用起来也不错，不过还是有一些瑕疵，现在的搜索是基于vim的pwd，而不是文件所在位置，像我这种人经常在vim里多开项目。这也简单，修改脚本，接受一个起始目录即可

### 版本三
```bash
#! /bin/bash
# a small script to find root
# the root is defind as .git or .svn or .root
# by linusboyle

if [ $# == 1 ];then
    if test -f $1;then
        base_dir=$(dirname $1)
    elif test -d $1;then
        base_dir=$1
    else
        exit 1
    fi
else
    base_dir=$PWD
fi

cd $base_dir

found="false"

function test_root(){
    ls -1a |egrep "^(.root|.git|.svn)+$" >/dev/null 2>&1 && found="true"
}

test_root

while [ $found == "false" ] && [ $PWD != "/" ]
do
    cd ..
    test_root
done

if [ $found == "true" ];then
    #absolute path
    pwd
else
    echo "not found"
fi
```

最终版：
```vim
" grep operator
nnoremap <leader>g :set operatorfunc=GrepOperator<cr>g@
vnoremap <leader>g :<c-u>call GrepOperator(visualmode())<cr>

function! GrepOperator(type)
    if executable('find_root')
        let saved_unnamed_register = @@
        let project_root=systemlist('find_root '.expand('%'))[0]

        if a:type ==# 'v'
            normal! `<v`>y
        elseif a:type ==# 'char'
            normal! `[v`]y
        else
            "ignore multiline mode,just because it's not useful
            return
        endif

        if l:project_root ==# 'not found'
            "search in current dir
            silent! execute "grep! " . shellescape(@@)
        else
            "else search in root dir
            silent! execute "grep! " . shellescape(@@) . " ". project_root
        endif

        copen
        let @@ = saved_unnamed_register
    else
        echo "missing find_root script,u reinstall the system again?"
        return 
    endif
endfunction
```
