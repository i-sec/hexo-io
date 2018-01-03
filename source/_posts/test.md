---
title: 记一道PWN题的解题思路
date: 2018-01-02 21:16:22
categories: 转载
tags:
- linux
- sec
- hack
toc: true
---

# 0×00前言
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=28285910&auto=1&height=66"></iframe>

一直以来都在搞逆向，没事破解点小程序，打打CTF。但是CTF上的逆向题也是越来越难了，各种套路让人防不胜防。都说漏洞利用是门艺术，于是就决定来学学pwn。

作为一名初学pwn的新人，在经过一段时间的“闭关”学习后，觉得是时候“出关”了，于是就从pwnable.tw上找来了一道200分的题applestore，由于比较“膨胀”，直接来200的，所以最后……但是pwnable.tw上的题还是不错的，有兴趣可以去尝试一下。

pwnable.tw不上外放高分题目的write up，但是我觉得200分也不算高分，还是可以分享下解题思路的，有什么不足之处欢迎批评指教。

![叮当猫](https://i.loli.net/2018/01/03/5a4c7b5ecc424.jpg)

# 0×01题目解析
这道pwn题给出了程序applestore与libc库文件。

运行程序可知这是个Apple商店，程序类似于一些note类型的题目，有添加、删除、查看、结算等功能。
![480531.png](https://i.loli.net/2018/01/03/5a4c7c12e3d5c.png)
# 0×02逆向分析
main函数
使用IDA打开程序进行逆向分析，首先来到main函数中。

<!--more-->

```
.text:08048CA6                 push    ebp
.text:08048CA7                 mov     ebp, esp
.text:08048CA9                 and     esp, 0FFFFFFF0h
.text:08048CAC                 sub     esp, 10h
.text:08048CAF                 mov     dword ptr [esp+4], offset timeout ; handler
.text:08048CB7                 mov     dword ptr [esp], 0Eh ; sig
.text:08048CBE                 call    _signal
.text:08048CC3                 mov     dword ptr [esp], 3Ch ; seconds
.text:08048CCA                 call    _alarm
.text:08048CCF                 mov     dword ptr [esp+8], 10h ; n
.text:08048CD7                 mov     dword ptr [esp+4], 0 ; c
.text:08048CDF                 mov     dword ptr [esp], offset myCart ; s
.text:08048CE6                 call    _memset
.text:08048CEB                 call    menu
.text:08048CF0                 call    handler
.text:08048CF5                 leave
.text:08048CF6                 retn
```

在main函数中，memset函数为全局变量初始化了一块大小为16个字节的空间，地址为0x804B068。

menu函数为显示的菜单，handler函数是程序的主要内容。
