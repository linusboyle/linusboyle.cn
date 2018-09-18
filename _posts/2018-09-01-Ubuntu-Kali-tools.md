---
title: Ubuntu Update Kali-Tools
date: 2018-09-01
tags: Linux
layout: post
---
### 创建shell文件
>vim update-kali-source.sh

```shell
cp /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib" > /etc/apt/sources.list
echo "deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib" >> /etc/apt/sources.list
apt-get update
#check 
#apt-key adv --keyserver keyserver.ubuntu.com --recv-keys #QianMing
apt-get install -y metasploit-framework && apt-get install -y nmap  && apt-get install -y zenmap && apt-get install -y armitage && apt-get install -y burpsuite && apt-get install -y beef && apt-get install -y dnswalk && apt-get install -y nbtscan && apt-get install -y netcat && apt-get install -y xprobe2 && apt-get install -y dhcping && apt-get install -y wpscan && apt-get install -y sqlmap && apt-get install -y wordlists && apt-get install -y mimikatz && apt-get install -y winexe && apt-get install -y hashcat && apt-get install -y johnny && apt-get install -y ophcrack && apt-get install -y hydra && apt-get install -y medusa && apt-get install -y john && apt-get install -y aircrack-ng && apt-get install -y wifite && apt-get install -y driftnet && apt-get install -y ettercap-g* && apt-get install -y ettercap-t* && apt-get install -y wireshark && apt-get install -y binwalk
echo "ok!"
cp /etc/apt/sources.list.bak /etc/apt/sources.list
apt-get update
```
