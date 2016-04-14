---
layout:     post
title:      "Android设备连接Ad-HOC网络"
date:       2011-02-11 15:53
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - Ad-HOC
---

如果你所在的地方有有线网络，但是没有无线信号，而你的笔记本又不支持虚拟Access Point（如果支持的话你还有一个选择就是connectify，尽管有人反映它不太稳定），你的Android设备又希望通过Wifi上网，怎么办呢？本文介绍的方式希望可以帮你解决这个问题。 :)

---## 本文所介绍的方法适用于：1. Android机器获得root权限，在2.1的Motorola Milestone上此方式正常工作。2. 笔记本带WiFi设备并安装的是Windows 7操作系统。3. 笔记本可以通过有线的以太网口上网。
---## 笔记本设置：1. Windows设置
	Control Panel -> Network and Internet -> Network and Sharing Center(网络和共享中心)点击Set up a new connection or network(建立一个新连接或者网络),然后选择最后一项“Set up a wireless ad hoc(computer-to-computer) network”，依据提示配置wifi名称，密码等，这里加密类型我们选择WEP。并且无线网卡的安全模式配置成“共享”（Shared）。2. 以太网设备的共享设置配置为允许WiFi设备通过它连接到Internet。3. 经过以上配置笔记本网络已经准备完成了，在右下角的网络连接中可以将他设置为“等待用户链接”即可。---
## 手机端配置：1. 修改文件前建议事先备份，还原的时候注意还原文件读写权限。2. 编辑/system/etc/wifi/tiwlan.ini（此文件是root用户可读写）:查找WiFiAdhoc = 0，并把0改成1，在这行下面追加两行内容：    `/system/etc/wifi/tiwlan.ini`
	    dot11DesiredSSID = 你的adhoc热点名	    dot11DesiredBSSType = 03. 编辑/data/misc/wifi/wpa_supplicant.conf（此文件是wifi用户可读写），开头原本就有两行，自己补充完整使开头的几行变成一下的形式：
    `wpa_supplicant.conf`
            update_config=1		ctrl_interface=tiwlan0		eapol_version=1		ap_scan=2		fast_reauth=1    如果你配置了WEP密码，则追加以下内容，注意引号不要漏掉。    `wpa_supplicant.conf`
    		network={			ssid="你的adhoc热点名"			scan_ssid=1			mode=1			key_mgmt=NONE			wep_key0="你的WEP密码"			auth_alg=SHARED		}	如果你没有配置密码，则追加以下内容，同样引号不能漏掉。	`wpa_supplicant.conf`
		network={			ssid="你的adhoc热点名"			scan_ssid=1			mode=1			key_mgmt=NONE		}4. 手机无线网络配置，选择高级配置，自己设定无线网络参数。	4.1. 网关填入希望使用的静态IP地址，注意这个地址要和笔记本的以太网卡处于不同的网段内。		4.2. 如果你的以太网卡的地址是192.168.1.xxx，那么你可以选择192.168.0.xxx等IP地址。		4.3. 子网掩码配置成255.255.255.0		4.4. 网关不需要配置，或者尝试IP地址网段的首个IP地址，例如IP网段为192.168.0.11，则网关可以尝试配成192.168.0.1		4.5. DNS可以填写Google的DNS服务器地址8.8.8.8和8.8.4.4，或者Open DNS服务器地址。然后重新启用网络看看是否可以正常连接，ping一下是否能连通网关。正常情况下经过以上步骤，你的Milestone应该已经可以通过笔记本共享网络上网了。