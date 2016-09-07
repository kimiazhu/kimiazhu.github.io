---
layout:     post
title:      "TinyServer - 全平台便携文件服务器"
date:       2016-09-07 19:30
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Golang
    - File Server
    - Http Server
---

### 目录：
{:.no_toc}
* any list
{:toc}

### 场景

[TinyServer](https://github.com/kimiazhu/tinyserver) 是一个用Golang实现的便携文件服务器。

TinyServer主要应用于这样的场景，在内网希望分享一些文件给某些朋友同事，或者需要PC上的一个文件传到手机上，等等之类的文件分享场景。手机设置共享太过麻烦，而且Samba之类的方案也有可能因为局域网安全策略之类的问题连接失败。

当然后者（PC往手机发文件）可以通过QQ的方式，但最近我在一场场景下发现QQ行不通，那就我在Linux上用openssl生成了证书，并希望将此证书文件在iOS上安装，用QQ接收文件后发现无法用浏览器打开并安装。实际上要顺利安装证书，必须使用浏览器打开此文件，这时候最直接的一个方式就是本地起一个服务器，手机处于同一内网通过URL下载并安装此证书。

---

### 使用

TinyServer使用非常傻瓜化，下载对应平台的可执行文件，并将它放入你要分享的文件夹下，即可通过8888端口进行文件的访问和下载。

---

### TODO

1. 提供两种模式：

  - 查看模式。对于浏览器能识别的文件，比如文本文件，直接在浏览器内提供内容展示。
  - 下载模式。对于浏览器不可识别的文件类型，点击链接直接下载。并在每个文件后提供直接下载链接和展示二维码的图片的链接（方便手机端下载）

2. 将配置信息包含进可执行文件中，这个需要对GinWeb进行改造。

3. 提供命令行参数选项。

---

### 引用

- [TinyServer](https://github.com/kimiazhu/tinyserver)
