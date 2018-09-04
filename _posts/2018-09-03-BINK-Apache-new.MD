---
title: CentOS6.5 Apache的增值服务
date: 2018-09-03
tags: Centos6.5服务器搭建
layout: post
---

### 1.更改Apache的监听端口号
>[root@localhost www]# vim /etc/httpd/conf/httpd.conf

```
#修改Listen后面的端口号为8008
Listen 8008
```
### 2.拒绝指定IP访问本网站
```
[root@localhost www]# service iptables start      //启动iptables
[root@localhost www]# iptables -L     //列出iptables的现有规则
[root@localhost www]# iptables -F     //清空iptables的所有规则
[root@localhost www]# iptables -A INPUT -s 192.168.150.110 -t tcp --sport 8008 -j ACCEPT      //-A 添加，INPUT 入站，-s 源地址，-p 协议，--sport 源端口，-j 动作，ACCEPT 通过
```
### 3.配置.htaccess对web服务目录启用用户认证
###### 1) 对需要认证的资源所在的目录进行配置
> [root@localhost www]# vim /etc/httpd/vhost/vhost.conf

```
＜Directory "/var/www/html"＞
　　Options Indexes FollowSymLinks
　　AllowOverride authconfig      //authconfig 表示对/var/www目录下的内容进行用户认证
　　Order allow,deny
　　Allow from all
＜/Directory＞
```
###### 2) 在限制访问目录`/var/www`下创建文件 **.htaccess**
> [root@localhost www]# vim /var/www/.htaccess

```
AuthName "提示信息"
AuthType basic
AuthUserFile  /home/members.txt    
require valid-user
```
###### 2.1) **.htaccess** 文件中常用的配置选项有一下几个：

  1) **AuthName**：定义提示信息，用户访问时提示信息会出现在认证的对话框中
　2) **AuthType**：定义认证类型，在HTTP1.0中，只有一种认证类型：basic。在HTTP1.1中有几种认证类型，如：MD5
　3) **AuthUserFile**：定义包含用户名和密码的文本文件，每行一对　　4) **AuthGroupFile**：定义包含用户组和组成员的文本文件。组成员之间用空格分开，如：group1:user1 user2
　5) **require** 命令：定义哪些用户或组才能被授权访问。如：
　　require user user1 user2 (只有用户user1和user2可以访问)
　　requires groups group1 (只有group1中的成员可以访问)
　　require valid-user (在AuthUserFile指定的文件中的所有用户都可以访问)
### 4.利用 **htpasswd** 程序生成包含用户名和密码的文本文件
>[root@localhost www]# htpasswd -bc /home/nemeber.txt user1 123456

__只需第一次添加用户时加入-c选项__
