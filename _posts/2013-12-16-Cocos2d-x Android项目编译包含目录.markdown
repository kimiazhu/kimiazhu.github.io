---
layout:     post
title:      "Cocos2d-x Android项目编译包含目录"
date:       2013-12-16 01:28
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Games
    - cocos2d-x
    - android
    - C++
---

2.2中使用create_project.py脚本生成的工程在Mac下（我只试过Mac下）似乎使用了错误（不完整）的包含路径，这里有一份可用的路径到export文件。以及include路径的配置界面的截图。

具体使用可能要更改其中前缀部分，因为这里直接使用了绝对路径。

![Cocos2d-x Android项目编译包含目录](https://dl.dropboxusercontent.com/u/3175114/Articles/2013-12-16-Cocos2d-x%20Android%E9%A1%B9%E7%9B%AE%E7%BC%96%E8%AF%91%E5%8C%85%E5%90%AB%E7%9B%AE%E5%BD%95/include_path.png)

[下载我的包含文件示例](https://dl.dropboxusercontent.com/u/3175114/Articles/2013-12-16-Cocos2d-x%20Android%E9%A1%B9%E7%9B%AE%E7%BC%96%E8%AF%91%E5%8C%85%E5%90%AB%E7%9B%AE%E5%BD%95/CocosIncludePath.zip)。