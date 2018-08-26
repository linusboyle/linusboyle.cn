---
title: CentOS6.5 ISO镜像挂载，创建本地yum源
date: 2018-08-23
tags: Linux
layout: post
---


1.虚拟机挂载光盘选择相应的镜像
--------
2.@#￥%……&*
-------
```
mkdir /mnt/cdrom
mount /dev/cdrom/ /mnt/cdrom/
```
3.修改源文件
-------
>vim /etc/yum.repos.d/Centos-Media.repos

Media.repos是本地安装的文件
修改为
```
[c6-media]
name=CentOS-$releasever - Media
baseurl=file:///mnt/cdrom/        #对应第二步创建的路径
gpgcheck=0                        #软件包验证改为0即可跳过
enabled=1                         #改为1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

然后基本上就完成了
>yum clean all      #清除缓存

>yum makecache      #重建缓存可以让搜索包时速度更快



如果无法安装可以试着将`/yum.repos.d/`目录下除`Centos-Media.repos`外的文件删除。然后在`yum update`
