---
layout:     post
title:      "Moto Milestone无法发送中文短信"
date:       2012-04-13 11:25
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - android
    - milestone
---

老机器了，摩托里程碑一代（感觉质量一般，按键容易损坏，话说这款产品虽是摩托第一次做的Android手机，但不是第一次做手机吧？），刷机后无法发送中文短信，所有中文字都被删除了，发出去的只有英文字符、英文符号和数字等拉丁字符，找到两种解决方案，**第一种验证过**，第二种就没去验证了，做个记录。

---

## 修改default.prop文件

1. Root机器，现在milestone已经可以一键root了（可以试试UniversalAndroot.apk，请自Google之）。

2. 重新挂载system分区为可读写，moto上可以切换成root后用在终端执行以下命令：
	
	***mount -o remount,rw -t yaffs2 /dev/block/mtdblock1 /system***

3. 把/system/default.prop中的**sms.convert.char.for.latam=1**改成**sms.convert.char.for.latam=0**，更新方法可以直接vi（如果你机器装了vi的话），或者adb pull拉出来，修改完再adb push回去。（发现push的时候没有写权限？可以考虑先push到SD卡，然后**adb shell**连接到机器运行上面第二点提到的mount命令挂载分区为可读写状态，再**cp /sdcard/default.prop /system**来覆盖原文件吧。总之方法是人想的，机器已经root了，权限问题就没有解决不了的 :) ）
4. reboot and try again. :)

---

## 修改系统设置

这是论坛里的说法，个人并未验证过：1. 还是得root机器。2. 将/data/data/com.motorola.android.providers.settings/databases/settings.db拷贝出来3. 使用sqlite3修改里面的settings表的数据，将name字段为sms_force_7bit_encoding的值从1改成04. 重启，发段中文试试？**奇怪的是我的机器上并没有sms_force_7bit_encoding的数据，所以这种方式也无法验证，或许可以插入一条数据试试？**