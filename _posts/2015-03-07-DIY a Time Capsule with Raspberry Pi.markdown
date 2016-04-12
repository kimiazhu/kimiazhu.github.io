---
layout:     post
title:      "DIY a Time Capsule with Raspberry Pi"
date:       2015-03-07 18:10
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Raspberry Pi
    - Mac
    - OS X
    - Time Machine
    - Time Capsule
---

先说说使用一天的情况：

Time Capsule可以运行，正常进行备份，但是备份速度很慢，尤其是preparing阶段可能就得消耗个半小时，而且（可能是路由器的问题）容易断线，另外Pi还会时不时自动重启。我这边正常的情况下备份时通过wifi的传输速度是2M左右，( Model B+ 512MB RAM ) 。至今差不多备份了整整一天，还是是没备份完我的200G不到的内容（首次备份），中间断了好几次。

后来等得心塞，取下外接硬盘挂到电脑上准备先坐一个完整备份，失败了，OS X判断之前备份到一半的内容不能继续使用（我估计是权限的问题），说要删除并重新开始一个新德备份进程。我再挂回Pi上，Pi上也不认这个备份了（挂入Mac机的时候系统做了一些修改，明显看到文件名已经变化，并且图标上加了个锁，可能是设置只读了，因为就备份了28G，不想再去折腾了）于是直接回到Mac格式化分区，再挂回Pi中重新做备份（Time Machine也自动可以给你删掉之前的备份然后再重新创建新备份，但是花费时间相当长）。

目前已经备份估计1小时有余，只完成了3G不到的内容。一句话，Pi可以用做Time Capsule，前提是你有耐心！

---

## 如何做：

#### Step 0

先安装好系统，需要的话还可以配置好无线网卡。系统用的是官方推荐的Raspbian，2015-02-16-raspbian-wheezy版本。

#### Step 1

硬盘可以分区，我的750G硬盘分成2个区，一个100G格式化成exFat用来存一些零碎的东西，剩下一个150G用来做Time Capsule，直接在Mac机上格式化成扩展日志式。

#### Step 2

硬盘接入Raspberry Pi，注意如果红色灯闪烁，说明电力可能不足，sudo fdisk -l 命令发现硬盘没有被识别。此时需要外界电源线供电。

#### Step 3

安装必要软件。依次执行以下命令：

A) 先来更新下系统：

	sudo apt-get update

B) 安装软件，这里先一次性把需要的都装完了，后面可以直接用

	sudo apt-get install samba samba-common-bin hfsplus hfsutils hfsprogs ntfs-3g exfat-fuse netatalk avahi-daemon libnss-mdns

#### Step 4

挂载分区，因为我有多一个exfat分区，所以可以再安装上samba服务以便让windows机器也可以使用。这里一并处理掉：

创建挂载点并挂载磁盘：

	sudo mkidr /media/TimeCapsule
	sudo mount -t hfsplus -o force /dev/sda1 /media/TimeCapsule
	sudo chown -R pi:pi /media/TimeCapsule
	sudo mkidr /media/Backup
	sudo mount -t exfat /dev/sda3 /media/Backup

同时可以修改/etc/fstab，加入：

	/dev/sda2 /media/TimeCapsule hfsplus defaults,noatime,force 0 0
	/dev/sda3 /media/Backup exfat defaults,noatime 0 0

#####Step 5

配置并重启netatalk服务：

	sudo echo "/media/TimeCapsule \"Time Capsule\" options:tm" >> /etc/netatalk/AppleVolumes.default
	sudo service netatalk restart

#### Step 6

配置Anahi

A) 编辑nsswitch.conf

	sudo vim /etc/nsswitch.conf
	
在hosts:后添加"mdns"，添加后为：

	hosts:      files mdns4_minimal [NOTFOUND=return] dns mdns4 mdns

B) 编辑afpd配置文件：

	sudo vim /etc/avahi/services/afpd.service
	
添加如下内容：

	<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
	<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
	<service-group>
	    <name replace-wildcards="yes">%h</name>
	    <service>
	        <type>_afpovertcp._tcp</type>
	        <port>548</port>
	    </service>
	    <service>
	        <type>_device-info._tcp</type>
	       <port>0</port>
	       <txt-record>model=Xserve</txt-record>
	    </service>
	</service-group>

C) 重启服务

	sudo service netatalk restart
	sudo service avahi-daemon restart

#### Step 7

All Done! 现在Time Machine上应该可以看到一个位于raspberrypi上的Time Capsule了。
