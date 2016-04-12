---
layout:     post
title:      "Ubuntu配置Android真机设备调试"
date:       2013-06-21 11:34
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - adb
    - Linux
    - Ubuntu
    - Eclipse
---

# 1. 配置和搭建环境

搭建环境现在很方便了，装好java后，下载adb bundle解压缩就好。

# 2. 启用调试
但是调试的时候需要Linux识别Android真机设备，就是命令 *adb devices* 的时候要能识别，并且能够被Eclipse识别，需要做以下设置。

## 2.1. 在Android Manifest文件中声明应用为 *debuggable* 。

如果使用Eclipse时可以不设置，因为Eclipse部署运行App的时候会自动启用调试。
    
**在 *AndroidManifest.xml* 文件的 *<application>* 节点中，增加 *android:debuggable="true"*来开启调试。**
    
注意：如果你在*AndroidManifest.xml*中启用了调试，请记得在发布的时候关掉它！
    
## 2.2. 在Android设备上启用 *USB debugging* 。

1. 在Android3.2或者更老的机器上，可以在 *Settings > Applications > Development* 下找到调试选项。
    
2. 在4.0或者更高的设备上，在 *Settings > Developer options* 中。
    
注意：在4.2或者更新的设备上，Developer options默认隐藏了。要显示它，可以到 *Settings > About phone* 中连续点击 *Build number* 多次即可。
    
## 2.3. 让你的系统识别Android设备。

1. 如果你正在使用Windows，参见这个[OEM USB驱动](http://developer.android.com/tools/extras/oem-usb.html)文档安装驱动。
    
2. 如果你在使用mac，跳过这部，Mac不用做任何配置。
    
3. 如果你正在使用Ubuntu，你需要增加一个包含了设备的配置信息的 *udev* 规则文件，设备中，每个设备制造商有一个唯一的 *vendor ID* 作为标识，体现在 *ATTR{idVendor}* 属性中。这个ID可以从下面的 **USB Vendor IDs** 表格中获取。
        
   3-1) 创建：*/etc/udev/rules.d/51-android.rules* 文件。
       
   3-2) 直接参考我的文件吧：
       
        SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="0102", SYMLINK+="android_adb"
        SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", MODE="0666", GROUP="plugdev"
        SUBSYSTEM=="usb", ATTR{idVendor}=="0b05", MODE="0666", GROUP="plugdev"
        SUBSYSTEM=="usb", ATTR{idVendor}=="1004", MODE="0666", GROUP="plugdev"
        SUBSYSTEM=="usb", ATTR{idVendor}=="12d1", MODE="0666", GROUP="plugdev"
        SUBSYSTEM=="usb", ATTR{idVendor}=="0fce", MODE="0666", GROUP="plugdev"
        
 具体语法可以参考[writing udev rules](http://www.reactivated.net/writing_udev_rules.html)
     
   3-3) 执行 *chmod a+r /etc/udev/rules.d/51-android.rules* 设置文件可读。

    |Company | USB Vendor ID|
    |--------|--------------|
    |Acer|0502|
    |ASUS|0b05|
    |Dell|413c|
    |Foxconn|0489|
    |Fujitsu|04c5|
    |Fujitsu Toshiba	|04c5|
    |Garmin-Asus|091e|
    |Google|18d1|
    |Haier|201E|
    |Hisense|109b|
    |HTC|0bb4|
    |Huawei|12d1|
    |K-Touch|24e3|
    |KT Tech|2116|
    |Kyocera|0482|
    |Lenovo|17ef|
    |LG|1004|
    |Motorola|22b8|
    |MTK|0e8d|
    |NEC|0409|
    |Nook|2080|
    |Nvidia|0955|
    |OTGV|2257|
    |Pantech|10a9|
    |Pegatron|1d4d|
    |Philips|0471|
    |PMC-Sierra|04da|
    |Qualcomm|05c6|
    |SK Telesys|1f53|
    |Samsung|04e8|
    |Sharp|04dd|
    |Sony|054c|
    |Sony Ericsson|0fce|
    |Teleepoch|2340|
    |Toshiba|0930|
    |ZTE|19d2|

##2.4.关于华为手机的特殊情况：

华为手机在经过上面设置后，*adb devices* 会出现设备全是?????一串问号的情况，需要经过以下设置将机器模式选择成google模式。

1 使用电话拨打\*#\*#2846579#\*#\*

2 这时会出现一个菜单，选择 *projectMenu*

3 选择 *Background setting*

4 选择 *Usb port setting*
 
5 选择 *Google mode*

6 提示 *Successed, please restart handset to make it effect*

PS：在第五步中可以看到的模式包括

- Normal mode
- Google mode
- CTS mode
- Manufacture mode
- Authentication mode
- Other mode


# 3.参考

- [Using Hardware Devices](http://developer.android.com/tools/device.html)

- [USB Vendor IDs](http://developer.android.com/tools/device.html#VendorIds)