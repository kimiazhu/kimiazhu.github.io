---
layout:     post
title:      "Cocos2d-x基于Eclipse创建Android工程"
date:       2013-06-13 02:04
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d
    - Cocos2d-x
    - Games
    - Android
    - Eclipse
---

1. 在cocos2d-x框架下使用命令“./create-android-project.sh”创建项目
2. 在Eclipse下创建新项目，创建Android Project from existing code
3. 创建后，可能AndroidManifest.xml中ic_launcher.png是错的；另外项目properties中的library引用也有可能是错的，如果有需要请手动更改这两个地方
4. 新建builder，指定使用build_native.sh进行构建
5. 添加完整的CDT特性。File -> New -> Other -> C/C++ -> Convert to a C/C++ Project(Add C/C++ Nature)。选择当前工程和使用Android GCC编译器，点完成；如有需要再在C/C++ General -> Paths and Symbols中添加cocos2dx目录即可
6. 视情况在Application.mk中添加：APP_PLATFORM := android-8
7. 如果系统报告NDK_BUILD_PATH的问题（如果在cocos2d-x目录下建立的项目，应该不会有这个问题），请在C++中新建NDK_BUILD_PATH变量：/path/to/cocos2d-2.1rc0-x-2.1.2;/path/to/cocos2d-2.1rc0-x-2.1.2/cocos2dx/platform/third_party/android/prebuilt;