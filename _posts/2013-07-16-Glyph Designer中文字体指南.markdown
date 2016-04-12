---
layout:     post
title:      "Glyph Designer中文字体指南"
date:       2013-07-16 23:33
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d
    - Cocos2d-x
    - Glyph Designer
    - Games
    - Font
    - Design
---

通过本教程你可以学到如何使用Glyph Designer创建如[图1]所示漂亮的中文字体，本文介绍到的效果包括描边、填充、渐变和阴影。

![图1](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/1.jpg)

[下载工程文件](https://github.com/kimiazhu/cocos-playground/blob/master/GlyphDesignerChineseSampleProject.GlyphProject?raw=true)


## 1. 打开一个新的Glyph Designer文档。

当你打开Glyph Desigher的时候，应用程序已经默认帮你新建了一个文档出来了。你可以使用Command+N或者File -> New菜单来创建一个新的文档。如[图2]所示。

![图2](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/2.jpg)

## 2. 修改字型

如[图3]，默认情况下Glyph Designer会包含NEHE字符集，英语中所有的字母和最常用的符号基本都已经包含在其中。修改 *Included Glyphs* 框中的文字成你需要显示的文字。不用担心其中有重复文字的问题，Glyph Desigher会自动去重并重新排列。

尝试把以下文字复制粘贴到 *Included Glyphs* 中：

**诶比西迪伊艾弗吉艾尺艾杰开艾勒艾马艾娜哦屁吉吾艾儿艾丝提伊吾维豆贝尔维艾克斯吾艾贼德**

![图3](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/3.jpg)

## 3. 选择适当字体

如[图4]，Glyph Designer左面板列出了你系统中的所有字体。如果你需要使用额外的其他字体也可以通过菜单中的 *load font* 进行选择。我们这里使用 *Adobe Heiti STD* 这个黑体字体来进行本示例演示。如果你的系统中没有这个字体，可以尝试 *STHeiti* 或者从[http://www.fonts.com](http://www.fonts.com)之类的在线字体库中下载一个。

需要注意的是你应当选择一个包含了你需要用的所有文字的字体文件。然后，再Glyph Designer左下角选择字体大小为131.

|key|value|
|-----|---------|
|Font Width|Adobe Heiti STD|
|Font Size|131|
	
![图4](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/4.jpg)

## 4. 应用描边效果

打开 *Stroke* 面板并设置描边宽度 *Stroke Widht* 为1.6以及 *Mitre Limit* 为0。接着，设置 *Color Type* 为 *gradient* 以应用渐变效果，并双击颜色调整框的中间，以便在这个文件增加一个渐变过渡点。（关于过渡点的操作，双击颜色对话框可以增加一个过渡点，鼠标左键点击之后可以拖动该点以调整颜色过渡情况，双击该点或者拖住该点拉到颜色框以外可以删除它。）

现在，你应该有了三个颜色过渡点了。将最左边和最右边的两个设置颜色值为 RGB(25,25,25)，中间的一个设置为 RGB(255,255,255)，通过旁边圆圈形状角度调整旋钮，将梯度调整为127度。

|key|value|
|-----|---------|
|Stroke Width|1.6|
|Mitre Limit|0|
|Color Type|Gradient|
|Color Stops|RGB(25, 25, 25) at 0% <br/>RGB(255, 255, 255) at 50%<br/>RGB(25, 25, 25) at 100%|	

![图5](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/5.jpg)

## 5. 设置填充颜色

打开填充面板，确保颜色类型是Gradient，设置旋转角度为0，像之前一样我们要设置三个颜色过渡点，左右两个是RGB(61,101,118)，中间一个是RGB(122,200,225)。

|key|value|
|-----|---------|
|Color Type|Gradient|
|Rotation|0|
|Color Stops|RGB(61, 101, 118) at 0% <br/>RGB(122, 200, 225) at 50% <br/>RGB(61, 101, 118) at 100%|

![图6](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/6.jpg)


## 5. 调整阴影效果

我们需要调整字体内部的阴影效果以便让字体有下沉的视觉效果。打开阴影面板，设置阴影类型为 *inner* ，设置x=2，y=-3。设置 *Blur Radius*为4，以及 *Shadow color* 为RGB(12,23,27)

|key|value|
|-----|---------|
|Shadow Type|Inner|
|X Offset|2|
|Y Offset|-3|
|Blur Radius|4|
|Shadow Color|RGB(12,23,27)|

![图7](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/7.jpg)

## 6. 更改纹理地图

最后一步，我们通过 *texture atlas settings* 面板更改背景颜色为黑色。一定要注意的是，更改背景颜色只会影响到它的Glyph Designer中你看到的样子，这个颜色并不会被导出到png当中，png永远都是透明的背景。它的作用只是让你在设计字体的时候，能清除地看到这个字体在不同背景下的使用效果。

|key|value|
|-----|---------|
|Background Color|Black|

![图8](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-07-16/8.jpg)

## 7. 参考

- [Glyph Designer Tutorial - Chinese Characters](http://www.71squared.com/en/product/491/gd-tutorial-chinese)