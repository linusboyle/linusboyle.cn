---
title: CentOS6.5 SendMail服务安装与配置
date: 2018-09-04
tags: Centos6.5服务器搭建
layout: post
music-id: 456272749
---
**一件很尴尬的事情，虽然步骤就是这样，但我死活都能成功过:(**
### 1.安装SendMail服务
>[root@centos mail]# yum install sendmail sendmai-cf dovecot
### 2.修改配置文件
##### 1). 修改sendmail.mc主配置文件
>[root@centos mail]# vim /etc/mail/sendmail.mc

```
#52,53两行取消注释
TRUST_AUTH_MECH('EXTERNAL DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
define('confAUTH_MECHANISMS','EXTERNAL GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
#第116行Addr后改为0.0.0.0
DAEMON_OPTION('Port=smtp,Addr=0.0.0.0,Name=MTA')dnl         
#第155行域名改为自己的域名
LOCAL_DOMAIN('mail.jnjs.net')dnl
```
>[root@centos mail]# m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf

##### 2). **local-host-names** 加入域名
>[root@centos mail]# vim /etc/mail/local-host-names

```
#加入本机域名
mail.jnjs.net
```
##### 3). 修改access文件
>[root@centos mail]# vim /etc/mail/access

```
#加入一行
Connect:10.1.10.101         RELAY
```
>[root@centos mail]# makemap hash access.db < htaccess

##### 4).修改dovecot文件
>[root@centos mail]# vim /etc/dovecot/dovecot.conf

```
#取消注释
protocols = imap pop3 lmtp
```
>[root@centos mail]# vim /etc/dovecot/conf.d/10-mail.conf

```
#第25行取消注释
mail_location = mbox:~/mail:INBOX=/var/mail/%u
```
>[root@centos mail]# vim /etc/dovecot/conf.d/10-auth.conf

```
#第九行取消注释并把yes改为no
disable_plaintext_auth = no
```
### 4.修改dns解析文件
>[root@centos mail]# vim /var/named/named.z

```
#加入
@     MX      5     mail.da1sy.com.
```
### 5.创建用户
```
[root@centos mail]# useradd user01 -p user01
[root@centos mail]# useradd user01 -p user02
```
### 6.启动 **sendmail** , **dovecot**
```
[root@centos mail]# /etc/init.d/sendmail restart
[root@centos mail]# /etc/init.d/dovecot restart
```
