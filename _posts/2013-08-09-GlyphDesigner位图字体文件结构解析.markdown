---
layout:     post
title:      "Glyph Designer位图字体文件结构解析"
date:       2013-08-08 21:00
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - C++
    - Glyph Designer
    - Games
    - Font
---

Glyph Desiger是Cocos2d的框架中最常用的位图字体设计工具。Cocos2d中配合 **CCLabelBMFont** 使用起来非常简单。但是如何在其他地方使用呢？我们恰好要在Java上使用它生成的字体文件，这里对字体结果做一个整理。

一般生成文件有两个文件构成，一个 **.fnt** （或者 **.txt** / **.LUA** ,我们这里只涉及 **.fnt** ） 格式的控制文件。一个是 **.png** 的字体文件。

##控制文件结构

**.fnt** 格式的控制文件是一个纯文本文件，用来辅助描述 **.png** 字体图片中每个字体的位置信息，以便使用的时候能够准确地从字体文件中截取正确的区域来使用。

一个 **.fnt** 格式的文件大概是下面这个样子的：

    info face="Arial-BoldMT" size=37 bold=0 italic=0 charset="" unicode=0 stretchH=100 smooth=1 aa=1 padding=0,0,0,0 spacing=2,2
    common lineHeight=42 base=35 scaleW=128 scaleH=64 pages=1 packed=0
    page id=0 file="bitmap-font-sample-1.png"
    chars count=11
    char id=32     x=103   y=33    width=0     height=0     xoffset=0     yoffset=34    xadvance=10    page=0 chnl=0 letter="space"
    char id=48     x=86    y=2     width=19    height=29    xoffset=2     yoffset=5     xadvance=21    page=0 chnl=0 letter="0"
    char id=49     x=88    y=33    width=13    height=28    xoffset=3     yoffset=6     xadvance=21    page=0 chnl=0 letter="1"
    char id=50     x=46    y=33    width=19    height=28    xoffset=1     yoffset=6     xadvance=21    page=0 chnl=0 letter="2"
    char id=51     x=23    y=2     width=19    height=29    xoffset=1     yoffset=5     xadvance=21    page=0 chnl=0 letter="3"
    char id=52     x=2     y=33    width=21    height=28    xoffset=1     yoffset=6     xadvance=21    page=0 chnl=0 letter="4"
    char id=53     x=25    y=33    width=19    height=28    xoffset=2     yoffset=6     xadvance=21    page=0 chnl=0 letter="5"
    char id=54     x=2     y=2     width=19    height=29    xoffset=2     yoffset=5     xadvance=21    page=0 chnl=0 letter="6"
    char id=55     x=67    y=33    width=19    height=28    xoffset=2     yoffset=6     xadvance=21    page=0 chnl=0 letter="7"
    char id=56     x=65    y=2     width=19    height=29    xoffset=1     yoffset=5     xadvance=21    page=0 chnl=0 letter="8"
    char id=57     x=44    y=2     width=19    height=29    xoffset=1     yoffset=5     xadvance=21    page=0 chnl=0 letter="9"
    kernings count=1
    kerning first=49 second=49 amount=-2
        
文件的头4行都是用来描述字体文件的一些特性的。包括字体、字号，是否粗体、斜体等等。如果设置了padding或者spacing也会在这里有所体现。

在头部信息之后是具体每个文字的信息。每个字独占一行，如下：

    char id=54     x=2     y=2     width=19    height=29    xoffset=2     yoffset=5     xadvance=21    page=0 chnl=0 letter="6"
        
如你所见，这里提供了相当多的信息。每一行表示一个字，每一行都是 **char** 打头，其他属性如下：

- id: 这一行或者说是它所代表的字符的唯一标识。实际就是该字符的Unicode码。
- x: 字体所在的X坐标
- y: 字体所在的Y坐标
- width: 字体的宽度
- height: 字体的高度
- xoffset: 绘制字体时需要向右移动的像素值
- yoffset: 绘制字体时需要向下移动的像素值
- xadvance: 绘制字体后需要向右跳过的像素
- page: 如果分成多个png图片的话，这个属性表示对应哪张图片
- chnl: 颜色通道，如果使用颜色通道的时候会写入这个值
- letter: 字符本身，这个属性对于调试程序相当有用哦。

##参考

- [《What is a Bitmap Font？》](http://www.71squared.com/en/product/2295/gd-tutorial-what-is-a-bitmap-font "What is a Bitmap Font")
- [Font Awesome](http://fortawesome.github.com/Font-Awesome/ "Font Awesome")