---
layout:     post
title:      "Cocos2d-x Android项目编译包含目录"
date:       2013-12-16 01:28
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Games
    - Cocos2d-x
    - Android
    - C++
---

2.2中使用create_project.py脚本生成的工程在Mac下（我只试过Mac下）似乎使用了错误（不完整）的包含路径，这里有一份可用的路径到export文件。以及include路径的配置界面的截图。

具体使用可能要更改其中前缀部分，因为这里直接使用了绝对路径。

![Cocos2d-x Android项目编译包含目录](/attachments/2013-12-16/cocos_include_path.png)

[下载我的包含文件示例](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-12-16/CocosIncludePath.xml)。