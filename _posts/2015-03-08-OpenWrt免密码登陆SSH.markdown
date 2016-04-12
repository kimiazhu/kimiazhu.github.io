---
layout:     post
title:      "OpenWrt免密码登陆SSH"
date:       2015-03-08 22:44
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - OpenWrt
    - Router
    - Mi Router
    - 小米路由器
    - PandoraBox
---

设备：小米路由Mini

由于官方开启SSH的原因，在极路由和小米路由之间还是选择了小米路由Mini。

至今为止，小米路由Mini的新开发版的固件并不能正常被SSH，（主要还是由于签名的问题，新的固件签名该了，导致SSH工具无法正常执行，设备SSH过程中会出现红灯常亮而失败的情况。）要获得SSH权限请自行搜索旧版本。可以考虑0.4.x的开发版。

SSH之后可以刷入OpenWrt或者它的派生版PandoraBox。

需要实现免密码登陆，并不能在root目录下建立.ssh目录放入authorized_keys，而应当把id_dsa.pub放入/etc/dropbear/authorized_keys中。

在本机执行代码生成公钥，或者使用现有的公钥：

`bash code:`

	cd ~/ mkdir .ssh && cd .ssh
	ssh-keygen -t dsa
	
上传公钥到OpenWrt系统中：

`bash code:`

	scp ~/.ssh/id_dsa.pub root@192.168.1.1:/tmp
	ssh root@192.168.1.1
	cd /etc/dropbear
	cat /tmp/id_*.pub >> authorized_keys
	chmod 0600 authorized_keys