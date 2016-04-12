---
layout:     post
title:      "XCode、Cocos2d-x、CocosBuilder杂记"
date:       2013-08-13 20:00
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - iOS
    - Android
    - Games
    - XCode
    - C++
    - Eclipse
    - CocosBuilder
---

eclipse在Ubuntu下实在不稳定，三天两头崩溃一次。。受不了了。刚刚切到XCode环境进行编译，问题遇到不少。比如编译一直找不到文件就是一个问题，折腾了不少时间。。## 1. 关于XCode资源文件找不到如果在Resource中直接添加一个文件夹文件，它的图标是黄色的，这样是不可以正常读取到文件夹中的资源文件的。此时如果去查看XCode打出来的XAP包，会发现资源文件的结构是平坦的，都被展开了，原本的目录结构不存在了。解决办法是在 把资源以目录的形式加入xcode时, 选择"Create folder references for any added folders ", 而不是默认的" Create groups for any added folders ", 加入后,xcode左侧树状结构内显示的图标是蓝色而非黄色.## 2. 关于CocosBuilder无故删除文件这个问题我遇到过两次，相当严重，这次直接导致我要重做大量工作！此Bug非常让人恼怒！github上也有人报告了[这个bug](https://github.com/cocos2d/CocosBuilder/issues/386)，但是不知为何这么严重的问题却至今还没有被处理。如果你是从CocosBuilder3.0 alpha5之前的版本升级到alpha5的，用的工程文件是老的，貌似就可能出现。如果你打算发布ios或者html5的ccbi，会将目录完全清空，包括你的原始ccb布局文件和所有纹理、字体等等！上面链接中有人回复提到的解决方法对我也无用。但是我发现在alpha5中直接新建的工程，没有此问题，所有目前只好新建工程一个个页面和资源从老的工程中导过来。蛋疼。## 3. NDK\_MODULE\_PATH问题当不是在Cocos2d-x框架目录下构建Android游戏时，可能会出现这个bug，网上有很多解决方案，但是基本上都不顶用。错误信息参考如下：

    Android NDK: jni/Android.mk: Cannot find module with tag 'CocosDenshion/android' in import path        Android NDK: Are you sure your NDK_MODULE_PATH variable is properly defined ?        Android NDK: The following directories were searched:        Android NDK:             jni/Android.mk:19: *** Android NDK: Aborting.    .  Stop.
        
网上的其他解决方案包括设置Eclipse环境变量，设置系统环境变量，直接硬编码路径，直接拷贝框架源代码到游戏相关目录等等。经过试验后两种可用。但是最后一种方法显然非常笨拙，下下策。当然实际一定可以用的还有一个办法是直接将工程建到Cocos2d-x目录下。但是如果我们要将来要给XCode也用同一份代码来打包iOS版本，这个方法就不适合了。

直接将路径写入到Android.mk可以解决。方法如下：

	$(call import-add-path, /Users/kimia/Software/cocos2d-x-2.1.4) \
	$(call import-add-path, /Users/kimia/Software/cocos2d-x-2.1.4/cocos2dx/platform/third_party/android/prebuilt) \
	$(call import-module,cocos2dx) \
	$(call import-module,CocosDenshion/android) \
	$(call import-module,extensions)
                	
其中前两行就是我们需要加入的路径。个人的机器各异。更好的办法当然是将这两个路径的环境变量抽取出来。

另外，由于这时候的游戏项目路径不是在Cocos2d-x目录下，（看看你在XCode中创建工程的时候放哪儿了？）所以build_native.sh文件种的三个变量也要修改下：

	# ... use paths relative to current directory
	COCOS2DX_ROOT="/Users/kimia/Software/cocos2d-x-2.1.4"
	APP_ROOT="/Users/kimia/Development/Cocos2dGames/TestGame"
	APP_ANDROID_ROOT="/Users/kimia/Development/Cocos2dGames/TestGame/proj.android"