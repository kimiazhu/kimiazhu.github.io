---
layout:     post
title:      "Golang Garbage Collection Analysis"
date:       2016-04-26 20:05
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Golang
    - GC
    - Garbage Collection
---

Go到1.5之后的GC机制已经有了长足进展，而扫描-标记-清除算法应该也能够支持循环引用，但是今天尝试发现了对于指针类型的循环引用，发现内存并没有释放。

