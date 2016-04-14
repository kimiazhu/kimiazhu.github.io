---
layout:     post
title:      "Nook Simple Touch中文支持"
date:       2012-03-21 10:53
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - nook
    - EPub
    - Androiod
---

我的第一台电纸书。

Nook原生不支持EPub格式的中文电子书，PDF中文没问题，EPub英文也没问题，要看中文EPub，可以选择两种方式：修改EPub文件本身，替换系统字体。另外本文会备注下替换系统的英英字典为英汉字典的方法。

**注意：修改系统文件前请做好备份！**

#### 如何选择：

* 修改EPub本身：每本书都必须进行替换（当然你可以用CssSTAR批量替换），但是不需要Root，操作成本很低。

* 替换系统字体：一劳永逸，但是前提是机器必须Root，而且现在的第三方中文字体普遍体积都很大（系统的都是100KB左右，第三方字体都是10MB左右，100倍的差距）会造成系统加载电子书变慢。


---

## 不Root修改EPub
这种方式对于每一本书都要单独改，然后打开书本后，在Text菜单中勾选**Publisher Default**，这时候就能发现书本的中文能够正常显示了，这个字体和微软雅黑很接近。看起来还是比较舒服的。

#### 修改方式：
找到EPub底下的css文件，可能是main.css，也可能是stylesheet.css，还可能是其他，文件名不重要，只要正文中HTML引用过来就行了。将对应的css文件的内容替换成以下内容：

```css
	
@font-face {	font-family: "DroidFont", serif, sans-serif;	font-weight: normal;	font-style: normal;	src: url(res:///system/fonts/DroidSansFallback.ttf);}@font-face {	font-family: "DroidFont", serif, sans-serif;	font-weight: bold;	font-style: normal;	src: url(res:///system/fonts/DroidSansFallback.ttf);}@font-face {	font-family: "DroidFont", serif, sans-serif;	font-weight: normal;	font-style: italic;	src: url(res:///system/fonts/DroidSansFallback.ttf);}@font-face {	font-family: "DroidFont", serif, sans-serif;	font-weight: bold;	font-style: italic;	src: url(res:///system/fonts/DroidSansFallback.ttf);}body { 
	font-family: "DroidFont", serif;
}
```

---

### Root后修改系统字体
Root之后可以替换系统字体，这样新加进来的书就不需要更改字体了，只要选择了我们替换的字体，就可以直接阅读。

[这里](http://www.hi-pda.com/forum/viewthread.php?tid=889528)有一些为1.1固件制作好的字体可以下载，注意1.0.1和1.1是最好使用不同的中文字体库文件。1.0.1的可以[参考这里](http://www.hi-pda.com/forum/viewthread.php?tid=805533&extra=page=1&filter=type&typeid=77&page=1)。

#### 建议：

1. 尽量只选择一两个字体替换，这些字体文件都很大，会严重影响电子书的打开速度。

2. 替换的时候要把对应字体的普通字体、粗体、斜体、粗斜体都替换掉。

3. 替换之后记得修改权限为644，我这里发现默认拷入SD卡的字体文件权限是075，Owner反而是没有任何权限的。

---

### 英汉词典
从[这里](http://122.70.220.99/downloads/basewords.db)下载。

系统字典路径位于**/system/media/reference/**下，替换后修改权限为644即可。

---

### 关于WiFi连接问题
Nook 1.1 固件的WiFi连接是有问题的，断开之后很难连接上，Nook官方论坛也有很多用户抱怨。两种方式解决：

1. 如果你未root机器，直接升级1.1.2；或者Root过的但不嫌麻烦，可以恢复出厂设置然后升级1.1.2。可以解决问题。

2. 如果你Root了，并且不想恢复出厂设置，可以使用以前1.0.1的libnetutils.so文件直接替换掉当前文件来解决，我是在官方更新放出之前做的，所以只能使用这种方法，并没有发现耗电量增加的问题。（不过我也并不经常用Nook连接WiFi网络）

---

### 参考

* [玩转Nook2系列二：折腾总结](http://www.chanue.com/it/some-technique-of-nook-2/)
* [Nook 1.0.1 以下固件用的中文字库](http://www.hi-pda.com/forum/viewthread.php?tid=805533&extra=page=1&filter=type&typeid=77&page=1)
* [Nook 1.1 用的中文字库](http://www.hi-pda.com/forum/viewthread.php?tid=889528)，1.1.1和1.1.2应该也是可以用的（官方对1.1.1的定位是修复了WiFi连接问题，1.1.2说是小幅系统增强），不过我未验证。
* [Nook2折腾笔记，折腾之后，是完美](http://www.hi-pda.com/forum/viewthread.php?tid=884843)