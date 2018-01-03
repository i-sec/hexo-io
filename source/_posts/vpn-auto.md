---
title: VPN断线后自动断网脚本
tags:
- VPN
- 匿名
- bash
- python
toc: flase
date: 2017-01-13 22:50:23
description:
categories: Tools
---
**实现功能:**

拨入vpn后,如果vpn掉线,将不能访问任何网络,以防暴漏自己真实ip地址

Windows :

1. python脚本(检测网卡信息如果没有ppp信息,释放网卡ip地址)
```py
#!#/usr/bin/env python
#coding=utf-8
import msvcrt 
import time
import os
x=0
print "Enter 'q' to exit"
vpn=os.popen("ipconfig |find \"PPP\"").read()
if vpn=="":
    print "No VPN"
else:
    x=1
while x:
    key=""
    time.sleep(0.5)
    if msvcrt.kbhit():
        key=msvcrt.getch()
    vpn=os.popen("ipconfig |find \"PPP\"").read()
    if(vpn==""):
        os.popen("ipconfig /release").read()
        print "Vpn Disable!!"
        opt=raw_input("Network is Down. Need up?(y/n):")
        if opt=='y':
            os.popen("ipconfig /renew").read()
            print "Network is Up!!!"
        exit(1)
    else:
        print "%d check: VPN OK!"%(x)
    x+=1
    if key=="q":
        print "Exiting..."
        exit(1)
        #os.system("ipconfig /renew")
```
编译后exe程序下载(适用无python环境)

2. 路由表操作

拨入vpn或者其他vpn客户端软件后,再本地路由表(route print) 可以看到两个网关信息,一个是本地网关记录,另一个是vpn网关记录
需要管理员权限来运行,可以自制bat

删除本地网关为192.168.1.1的默认路由(只通过vpn网关上网)

```bat
route delete 0.0.0.0 192.168.1.1
```
添加本地网关为192.168.1.1的默认路由(恢复网络)
```bat
route add 0.0.0.0 mask 0.0.0.0 192.168.1.1
```

**Linux(bash脚本)**
```bash
#!/bin/bash
n=1
while true;do
    ifconfig|grep ppp >/dev/null
    if [ $? -eq 0 ];
    then
        echo "$n Check: VPN OK"
        sleep 1
        n=`expr $n + 1`
    else
        ifconfig eth0 down
        echo "Network is Down"
        echo "please run:ifconfig eth0 up"
        exit
    fi
done
```

