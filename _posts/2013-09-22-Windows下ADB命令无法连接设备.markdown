---
layout:     post
title:      "Windows下ADB命令无法连接设备"
date:       2013-09-22 00:33
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - adb
---

Windows下无法连接Android设备，Eclipse下无法找到设备或者命令行下输入adb命令无法连接设备，输出如下：

	C:\Users\Kimia>adb devices
	adb server is out of date.  killing...
	ADB server didn't ACK
	* failed to start daemon *
	error: unknown host service

很可能的原因之一是端口被暂用，如果你装了一些手机管理软件，这个问题发生的概率相当大（我基本很少装这样的管理软件，目前遇到过的两个，分别是“金山快盘”里面带了个豌豆荚，还有QQ音乐）。

可以通过以下方式定位：

	C:\Users\Kimia>netstat -a -o 5037|grep 5037

得到类似输出：

	 TCP    127.0.0.1:5037         3dns-2:0               LISTENING       7644
 	 TCP    127.0.0.1:5037         acdid:49321            ESTABLISHED     7644
     TCP    127.0.0.1:5037         acdid:64336            TIME_WAIT       0
     TCP    127.0.0.1:5037         acdid:64337            TIME_WAIT       0
     TCP    127.0.0.1:5037         acdid:64344            TIME_WAIT       0
     TCP    127.0.0.1:5037         acdid:64345            TIME_WAIT       0
     TCP    127.0.0.1:49321        acdid:5037             ESTABLISHED     4708

发现第一条，PID为7644的进程占用了adb调试端口5037。这个必然出问题了。继续看看这个进程：

	C:\Users\Kimia>tasklist /fi "pid eq 7644"

	Image Name          PID Session Name        Session#    Mem Usage
	============== ======== ================ =========== ============
	tadb.exe           7644 Console                    1     23,124 K

或者可能是：

	C:\Users\Kimia>tasklist /fi "pid eq 7644"

	Image Name                 PID Session Name        Session#    Mem Usage
	===================== ======== ================ =========== ============
	wandoujia_daemon.exe      7644 Console                    1     23,124 K

基本就可以确定这个进程了，然后到任务管理器或者用命令杀死进程，再重试即可。

	taskkill /F /pid 4552