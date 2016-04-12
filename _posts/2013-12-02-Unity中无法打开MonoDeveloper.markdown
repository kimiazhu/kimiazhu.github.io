---
layout:     post
title:      "Unity中无法打开MonoDeveloper"
date:       2013-12-02 20:22
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - Windows Phone
    - WinRT
    - Visual Studio
    - Unity3D
---


- 操作系统：Windows 8.1 Pro
- Unity版本：4.3.1f1
- 问题：点击脚本无法用MonoDeveloper打开，直接打开MonoDeveloper也无效，Loading界面启动到大概50%时消失，进程也无法找到。
- 解决：下载[这个文件](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-12-02/glibsharpglue-2.dll)，替换 **Unity\Monodevelop\bin** 下的同名文件。

### 参考

1. [Monodevelop not opening in Unity 4.3](http://answers.unity3d.com/questions/574157/monodevelop-not-opening-in-unity-43.html)