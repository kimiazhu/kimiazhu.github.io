---

Date: 2015-03-07 18:00

Title: Raspberry Pi配置无线网络

Tags: Raspberry Pi, Linux

---

设备：Raspberry Pi Model B+ 512M

无线网卡：EDUP EP-N8508GS 免驱动

前期制作系统在官方教程中已经有很详细步骤了。大致如下：

1. 制作好TF卡，插入机器的TF卡槽
2. 网线插入以太网口
3. 插入电源启动机器
4. 在路由器上的地址列表中查看客户端名称为：raspberrypi 的机器的IP地址
5. 在电脑终端使用ssh连接：ssh pi@you-ip-address 密码是raspberry

---
###配置无线网卡

由于EDUP无线网卡无需驱动即可被Raspberry识别，以下直接开始无线网卡配置。

#####1. 配置interfaces：

`sudo vim /etc/network/interfaces`

	auto lo

	iface lo inet loopback
	iface eth0 inet dhcp
	
	#DHCP configuaration
	allow-hotplug wlan0
	auto wlan0
	iface wlan0 inet dhcp
	pre-up wpa_supplicant -B -Dwext -iwlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
	post-down killall -q wpa_supplicant

	#static ip configuaration
	#allow-hotplug wlan0
	#iface wlan0 inet manual
	#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	#iface default inet static
	#address 192.168.3.111
	#netmask 255.255.255.0
	#gateway 192.168.3.1
	#dns-nameservers 192.168.3.1 210.21.196.6 221.5.88.88

#####2. 配置wpa_supplicant.conf

`sudo vim /etc/wpa_supplicant/wpa_supplicant.conf`

	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	update_config=1
	ap_scan=2

	network={
		ssid="MySSID"
		proto=WPA2
		key_mgmt=WPA-PSK
		pairwise=CCMP TKIP
		group=CCMP TKIP
		#psk="test_password"
		psk=0d1f10ae43c320c2b8f548e9ad6ab0c7e78c4a91ac417ca123c21860cab3e12a
	}

其中，psk密码，是加密过后的密码而不是原始密码，加密方法：

	wpa_passphrase [ssid] [password]

*PS：如果使用DHCP配置，还可以在路由器端设置保留IP地址，分配给该网卡，以便每次都能以相同的IP地址访问该机器。*

#####3. 使用以下命令重启无线网络：

	sudo ifdown wlan0
	sudo ifup wlan0 -v # 加V可以看到具体启动过程

#####4. 参考资料：

-  [官方文档](http://www.raspberrypi.org/documentation/)
- [Raspberry Pi usb 无线网卡安装配置](http://www.chinaxing.org/articles/linux/2013/03/08/2013-03-08-raspberry-pi-usb-wifi-config.html)
- [Raspberry Pi树莓派无线网卡配置[多重方法备选]](http://blog.appdevp.com/archives/188) 这篇文章里的动态IP的第二种配置方法
