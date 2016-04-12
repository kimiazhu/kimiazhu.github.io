---
layout:     post
title:      "Ubuntu中解决远程桌面无法使用ctrl键的问题"
date:       2013-06-13 01:55
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Linux
    - Ubuntu
    - Remote Desktop
---

出现这个问题是因为设置了“按下Ctrl键时显示鼠标位置”引起的，因为勾选了该设置之后，Ubuntu会捕捉ctrl事件，并且不会再传递给远程桌面客户端或者虚拟机。

但是到了ubuntu 13之后，这个设置在面板上被取消了，这时候只能从命令行或者使用dconf-tools进行设置。如下：

*This might or might not, work.*

1. Install Dconf editor:

    `Shell Command: `
    
        sudo apt-get install dconf-tools

2. Open Dconf and navigate to:

    *org -> gnome -> settings deamon -> peripherals -> mouse*
    
    There should be a setting there titled  *locate-pointer* .
    
![Dconf配置](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-13/1.jpg "Dconf配置")

If it's not there, then I don't think its possible to change this. Very weird it defaults to on...

在Ubuntu中，以上设置可以基于系统首选项中的鼠标和觸控板配置页面完成，但是在Ubuntu13.04开始，已经无法这么做了，只能通过上面的DConf的方式进行。

原因是与 Ubuntu 12.10 版本比较，13.04中去掉了不少东西，所以也就直接导致无法从GUI上进行设置会恢复上面步骤提到的过程：

![Ubuntu12的鼠标和觸控板配置页](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-13/2.jpg "Ubuntu12的鼠标和觸控板配置页")
上图是 Ubuntu 12.10 中的鼠标和触控板设置


![Ubuntu 13.04 中的鼠标和触控板设置](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-13/3.jpg "Ubuntu 13.04 中的鼠标和触控板设置")
上图是Ubuntu 13.04 中的鼠标和触控板设置

---
PS:

以前一直用RH和Fedora系列，这是第一次用Ubuntu（当然不是第一天），总体感觉是，配置更容易了，驱动更方便了。但是稳定性大大降低。。

我是不想浪费时间在系统安装和维护上，才选择了Ubuntu的，但是稳定性确实让人泪奔。。