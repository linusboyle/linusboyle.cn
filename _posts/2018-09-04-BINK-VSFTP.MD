---
title: CentOS6.5 Vsftp服务安装与配置
date: 2018-09-04
tags: Centos5.5服务器搭建
layout: post
---

### 1.安装vsftp服务
>[root@localhost ~]# yum install vsftpd

### 2.软件结构
- **/etc/vsftpd/vsftpd.conf**
    - vsftp的主配置文件，“参数=设置值”。**等号两边不能有空白**。
- **/etc/pam.d/vsftpd**
    - 是vsftp使用PAM模块时的相关配置文件。主要用来作为身份认证。
- **/etc/vsftpd/ftpusers**
    - 主要是将禁止登录的账户写入进去（一行一个）
- **/etc/vsftpd/chroot_list**
    - 默认是不存在的，主要的功能是可以将是将实体用户限制在他们的用户主目录下而不许到其他目录。但这个文件要生效与vsftpd.conf内的 **chroot_list_enable**、 **chroot_list_file** 两个参数有关
- **/var/ftp/**
    - 默认匿名用户登陆的根目录

### 3.主要配置参数说明
- **anonymous_enable=YES**
默认是允许匿名用户登录，禁止改为NO
- **#chroot_list_enable=YES**
不允许FTP用户离开自己的主目录，默认是被注释的
- **#chroot_list_file=/etc/vsftpd/chroot_List**
如果开启了 **chroot_list_enable=YES** 也一定要开启。是锁定登录用户只能在主目录的位置
- **local_enable=YES**
允许本地用户访问，默认是yes
- **wirte_enable=YES**
允许写入。默认是yes
- **local_umask=022**
上传后文件的权限掩码
- **dirmessage_enable=YES**
开启目录标语
- **xferlog_enable=YES**
开启日志
- **connect_from_port_20=YES**
连接端口为20
- **xferlog_std_format=YES**
设定vsftp的服务日志保存路径
- **#ftpd_banner=Welcome to blah FTP service**
ftp的登录欢迎语
- **#chroot_local_user=YES**
禁止本地用户登出自己的FTP主目录
- **pam_service_named=vsftpd**
设定pam服务下vsftpd的验证配置文件
- **userlist_enable=YES**
拒绝登录用户的名单
- **TCP_wrappers=YES**
限制主机对VSFTP服务器的访问（通过_/etc/hosts.deny_和_/etc/hosts.allow_这两个文件来配置）
- **guest_enable=YES**
设定启用虚拟用户功能
- **guest_username=ftpuser**
指定虚拟用户的宿主用户
- **virtual_use_local_privs=YES**
指定虚拟用户的权限符合他们的宿主用户
- **user_config_dir=/etc/vsftpd/vconf**
指定虚拟用户个人vsftp的配置文件存放路径

### 4.开始架设
#### 1). 修改主配置文件
>[root@localhost ~]# vim /etc/vsftpd/vsftpd.conf

```
//更改以下值
anonymous_enable=NO
//将96、97、99行取消注释
chroot_local_user=YES
chroot_list_enable=YES
# (default follows)
chroot_list_file=/etc/vsftpd.chroot.list
local_max_rate=1024
```
#### 2). 添加FTP用户并设置密码及组
```
[root@localhost home]# useradd -d /home/H1 -g ftp -s /sbin/nologin h1
[root@localhost home]# passwd h1
```
###### **-d /home/H1** 参数是为h1用户分配主目录，**-g ftp** 是将用户分配到ftp组，**-s /sbin/nologin** 是将其设置为系统用户但没有登录系统的权限
#### 3).启动服务
>[root@localhost home]#/etc/init.d/vsftpd start
