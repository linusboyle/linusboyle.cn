---
title: 自己动手写一个简易的tee
date: 2018-09-15
tag:
- linux
- tee
excerpt: 用glibc写一个简易的tee
layout: post
---
## tee

tee是脚本里很有用的工具（交互模式其实也很有用，不过我不怎么常用）。它会从标准输入读入，向标准输出和指定的文件输出

实现的思路很简单，不断地读取标准输入，write至标准输出和创建的文件标识符就可以。

不过这只是自己玩玩的，和GNU的tee还是差远了:)

## 实现

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>

#define MAX_BUFFER 1024

int main(int argc,char** argv){
    int fd,readbyte;
    char buffer[MAX_BUFFER];

    if(argc != 2){
        printf("Usage:mytee FILE\n"); 
        exit(EXIT_FAILURE); 
    }

    fd = open(argv[1],O_CLOEXEC|O_WRONLY|O_CREAT|O_TRUNC,S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH); 

    if(fd == -1){
        perror("open failed!");
    }
    
    while(1){
        readbyte = read(STDIN_FILENO,buffer,MAX_BUFFER);
        if(readbyte == 0){
            break;
        }
        if(write(STDOUT_FILENO,buffer,readbyte) == -1){
            perror("write to stdout failed!");
        }
        if(write(fd,buffer,readbyte) == -1){
            perror("write to file failed!");
        }
    }

    if(readbyte < 0){
        perror("read from stdin failed!");
    }

    close(fd);
    return EXIT_SUCCESS;
}
```
