---
title: CentOS6.5 配置Apache及多站点VirtualHost
date: 2018-08-29
tags: Linux
layout: post
---

1.安装httpd服务
-------
```
yum install httpd
```
2.配置文件
-------

Apache主要的配置文件是`/etc/httpd/conf/httpd.conf`,文件中包含许多在基本安装中不需要更改的配置。实际上只需要对文件稍微进行一些更改即可启动并运行服务。
2.1 监听端口
--------
Listen配置项，定义了Apache要监听页面请求的IP地址和端口。
```
Listen 80
#如果需要限制IP访问可以更改为
Listen 127.0.0.1:80
#即只可本地浏览
```
2.2 Html文件位置
--------
DocumentRoot指定组成该网站页面的HTML文件位置。默认为
```
DocumentRoot "/var/wwww/html"
```
3.主配置文件详解
-----------


无法访问HTTP服务器的可能情况：
---------------
1.查询Selinux的状态
```
[root@www vhost]# getenforce
```
2.临时关闭Selinux
```
[root@www vhost]# setenforce 0
#sentenforce 1 //为enforcing模式  即为开启状态
```

----------------------------
建立多站点的VirtualHost
==========================
### **1.创建站点文件目录**
```
[root@www vhost]# mkdir /var/www1
#第一个站点的文件目录
[root@www vhost]# mkdir /var/www2
#第二个站点的文件目录
```
### 2.配置主文件
>vim /etc/httpd/conf/httpd.conf

```
#添加
Include /etc/httpd/vhost/vhost.conf
#取消注释
NameVirtualHost *:80
```
### 3.创建虚拟主机配置文件
```
[root@www vhost]# mkdir /etc/httpd/vhost
[root@www vhost]# cd /etc/httpd/vhost
[root@www vhost]# vim vhost.conf
```
#### 写入如下内容:
```shell
<VirtualHost *:80>
   ServerName www.da1sy.com
#虚拟主机1的域名
  DocumentRoot /var/www1/
#虚拟网站目录
   DirectoryIndex index.html index.php
#默认首页
</VirtualHost>
<Directory "/var/www1/">
   Options +Includes -Indexes
   AllowOverride All
#允许所有请求
   Order Deny,Allow
   Allow from All
</Directory>

<VirtualHost *:80>
   ServerName www2.da1sy.com
   DocumentRoot /var/www2/
   DirectoryIndex index.html index.php
</VirtualHost>
<Directory "/var/www2/">
   Options +Includes -Indexes
   AllowOverride All
   Order Deny,Allow
   Allow from All
</Directory>

```
### 4.编辑`/etc/hosts`文件
>[root@www vhost]# vim /etc/hosts
#### 添加如下两行
```
192.168.150.11  www.da1sy.com
192.168.150.11  www2.da1sy.com
```
### 5.重启httpd
```
[root@www vhost]# service httpd restart
```
