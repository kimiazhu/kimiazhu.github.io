---
layout:     post
title:      "使用TexturePacker加密保护图片资源"
date:       2013-08-07 23:00
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - C++
    - TexturePacker
    - Games
---


前阵子有幸从[Texture Packer作者处](http://www.codeandweb.com/ "Code'n'Web")得到了SN一枚。特此感谢一下作者为我们提供了这么好的工具！

对于TexturePacker,我们之前使用中发现最大的不足莫过于所有图片都是无法加密的，导致别人拿到的iOS或者Android安装包，直接当成zip解压出来，就可以获取到其中的纹理资源。

现在这个问题已经改变了，从Version 3.0.9 (2013-04-19)开始，TexturePacker已经支持加密纹理。加密后的纹理直接解压出来是无法看到的，必须输入正确的密钥。而要在游戏中正确地使用这个加密的纹理，自然也需要配置密钥。

需要注意的是，目前只有 **pvr.ccz** 格式支持加密，如果你是用的jpg或者其他格式，那是没办法的。

Cocos2d的支持比Cocos2d-x早一部，不过目前的Cocos2d-x-2.1.4也已经支持上了。对于之前的Cocos2d-x版本不想升级或者无法升级可以尝试从2.1.4之后提取两个类： **ZipUtils.h** 和 **ZipUtils.cpp** 放入对应目录中。

Cocos2d-iphone版本的文档可以在作者页面上找到原文：《[TexturePacker Content Protection](http://www.codeandweb.com/blog/2013/04/19/texturepacker-content-protection "TexturePacker Content Protection")》 。

## 1.Cocos2d-iphone中使用内容加密

1. 在TexturePacker中确定要加密的纹理，首先确保格式是 **.pvr.ccz** 点击Content Protection后面的小锁标志。
![图1. 使用加密](/attachments/2013-08-08/1.jpg "图1. 使用加密")

2. 设置密钥。你可以将密钥保存成Global Key，以便如后使用。
![图2. 设置密钥](/attachments/2013-08-08/2.png "图2. 设置密钥")

3. 在代码中要正确使用加密过的文件，需要设置密钥。（确保你有新版本的cocos2d框架，或者下载这两个文件以便程序能够支持加密内容： **[ZipUtils.h](http://www.codeandweb.com/public/contentprotection/cocos2d/ZipUtils.h "ZipUtils.h")** 和 **[ZipUtils.m](http://www.codeandweb.com/public/contentprotection/cocos2d/ZipUtils.m "ZipUtils.m")** ）

   密钥必须是32位16进制数字，假设密钥是 *aaaaaaaabbbbbbbbccccccccdddddddd* （必须和你在TexturePacker中使用的相同哦！）你需要将它等分成四段，每段8位，然后在游戏开始使用任何纹理之前调用一下方法：
   
    `Objective-C Code:`
    
        #import "ZipUtils.h"
        /////////////////////////////////////
        ///////////Other Code.../////////////
        /////////////////////////////////////
        caw_setkey_part(0, 0xaaaaaaaa);
        caw_setkey_part(1, 0xbbbbbbbb);
        caw_setkey_part(2, 0xcccccccc);
        caw_setkey_part(3, 0xdddddddd);
        
4. OK, 后面的代码和以前一样了！

## 2.Cocos2d-X中使用内容加密

1. 在TexturePacker中确定要加密的纹理，首先确保格式是 **.pvr.ccz** 点击Content Protection后面的小锁标志。
![图1. 使用加密](/attachments/2013-08-08/1.jpg "图1. 使用加密")

2. 设置密钥。你可以将密钥保存成Global Key，以便如后使用。
![图2. 设置密钥](/attachments/2013-08-08/2.png "图2. 设置密钥")

3. 在代码中要正确使用加密过的文件，需要设置密钥。（确保你有的cocos2d-x-2.1.4或以上版本，或者从2.1.4中拷贝以下两个类 **ZipUtils.h** 和 **ZipUtils.cpp** ）

   密钥必须是32位16进制数字，假设密钥是 *aaaaaaaabbbbbbbbccccccccdddddddd* （必须和你在TexturePacker中使用的相同哦！）你需要将它等分成四段，每段8位，然后在游戏开始使用任何纹理之前调用一下方法：
   
    `C++ Code:`
    
        #include "../cocos2dx/support/zip_support/ZipUtils.h"
        /////////////////////////////////////
        ///////////Other Code.../////////////
        /////////////////////////////////////
        ZipUtils::ccSetPvrEncryptionKeyPart(1, 0xaaaaaaaa);
    	ZipUtils::ccSetPvrEncryptionKeyPart(3, 0xbbbbbbbb);
    	ZipUtils::ccSetPvrEncryptionKeyPart(2, 0xcccccccc);
    	ZipUtils::ccSetPvrEncryptionKeyPart(0, 0xdddddddd);
    	
4. OK, 后面的代码和以前一样了！