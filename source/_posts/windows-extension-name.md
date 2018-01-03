---
title: Windows文件扩展名欺骗 
tags:
- Windows
- 社会工程学
- WinRaR
- RTLO
- RLO
toc: flase
date: 2016-01-09 22:56:02
description:
categories: 社会工程
---
**0x00  隐藏的扩展已知文件类型**

Windows默认的设置是隐藏已知文件类型的扩展名，如果开启了，可以在文件夹选项中开启已知扩展名的显示，当扩展名被隐藏后，如果有一个双扩展名，不太细心的用户就有可能会被欺骗。下面演示下双扩展：

notes.txt.exe

上面的文件实际上是一个可执行文件，但默认在Windows中却显示为notes.txt，那是因为隐藏了已知后缀，将exe隐藏了起来，我们为了让文件看起来更有说服力，可以将文件的图标更改为记事本图标，你可以从下面的示例图像中看到，它看起来就像一个普通的文本文件。

![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb51_thumb.png)
但是我们将视图显示为详细信息的时候，就可以看到它的属性是一个可执行文件
![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb71_thumb.png)
这个一个比较老的技巧了，一些防火墙或者杀毒软件会警告双后缀

防御： 显示已知后缀

**0x01 RTLO/RLO (right to left override)**

RTLO是Right to Left Override的缩写，他是一个 U+202E 的Unicode字符，用于阿拉伯语和希伯来语中将句子倒过来显示（从右往左显示），它可以让字符后面紧跟的字符串倒

可以用来欺骗用户打开可执行文件（钓鱼攻击），或者欺骗后端应用的检查机制。

虽然这个技术是有点老了，但仍然被像Etumbot，Sirefef等后门程序使用

三种操作的方法

这里我用cmd.exe 来测试

1. 打开字符映射表（可在开始菜单 - >附件 - >系统工具 中找到）：

![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb91_thumb.png)
选中 高级查看（Advanced view） ，在 转到Unicode （Go to Unicode） 中输入 202E回车后，自动跳到U+202E字符，点击选择（select）然后复制（select）

我们将 cmd.exe 重命名为  TESTcod.exe ,将复制的字符粘贴到TEST和cod之间，文件就变成了TESTexe.doc，应用后文件就更改好了

**格式： 
[文件名] [插入U + 202E字符] [反向的后缀，例如cod反向则为doc] [真正的扩展]**
![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb111_thumb.png)
![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb131_thumb.png)

最后效果图

![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb141_thumb.png)

还有一种方法：

直接在文件名中 TEST和cod之间插入 Unicode控制字符 ->RLO字符 即可,效果和上面一种方法一样

![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb161_thumb.png)
再贴上一个python脚本
```py
#!/usr/bin/env python 
import os
import sys
def exploit():
    name = sys.argv[1].split(".")[0]
    extension = sys.argv[1].split(".")[1]
    newname = os.getcwd()+os.sep+name+u"\u202E"+sys.argv[2][::-1]+"."+extension
    try:
        os.rename(sys.argv[1], newname)
        if os.path.isfile(newname):
            print "\n\nFile extension spoof complate !\n"
    except Exception as error:
        print "\nUnexpected error : %s" % error
    
exploit()
```

使用说明： 脚本 文件 伪造的后缀

![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb181_thumb.png)
RTLO只是混淆了人的视觉而已，但是系统还是识别为应用程序的，所以邮件中也会过滤的

防御：

(1) 注册表 HKEY_Current_User/Control Panel/Input Method下新增字串值EnableHexNumpad=1

(2) 点选”开始”→”执行”→输入”gpedit.msc”

(3) 点开”计算机配置”→”Windows设置”→”安全设置”

(4) 在”软体限制策略”上点选右键→”创建软件限制策略” (如果之前有设过别的软体限制策略，此步骤可忽略)

(5) 点开”软件限制策略”→在”其他规则”上点选右键→”新增路径规则”→在”路径”处输入”\*[202E]\*”(注1)，安全性等级= ”不允许”→”确定”

(6) 重新开机

注1：[202E]的输入方式为长按[Alt]，依序输入[+], [2], [0], [2], [E]，注意路径处前后需加上\*.

参考：http://v00d00sec.com/2015/05/05/file-extension-trick-using-rtlo-u202e/

**0x02 软件漏洞**

例如老版本的 WinRAR 4.20 创建的ZIP文件，可以用十六进制编辑器修改HEX码，修改窗口中显示的文件后缀，如果你习惯从压缩包中直接打开图片或者文档，那会是致命的，但是如果文件解压后，显示的还是exe，可以配合RLO，效果更好哦

修改第二个显示的文件名，一般在文件最后，修改完成后保存即可，不过解压后还是会显示cmd.exe

![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb201_thumb.png)

配合RLO效果，修改图标就基本完美了
![](http://of3vvgi8i.bkt.clouddn.com/images/Spoofing_9EDS/6da8ea67509b_3CF0/image_thumb221_thumb.png)

参考：https://www.exploit-db.com/papers/32480/

防御：升级你的WinRAR



<!-- more -->
