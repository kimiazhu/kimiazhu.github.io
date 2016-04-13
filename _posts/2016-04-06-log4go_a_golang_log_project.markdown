---
layout:     post
title:      "log4go: A Golang Log Project"
date:       2016-04-06 20:05
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Golang
---

# 1. 说明

现有的golang标准库中的log方案不支持滚动，也不支持多个日志级别输出，使用起来会有些不方便，而网上也有不少的针对go的log解决方案。我这个log方案基于[alecthomas/log4go](https://github.com/alecthomas/log4go)改造而来。并且已经改过很多东西，所以讲它单独独立出来了。目前我们自己的生产环境已经在使用。

核心结构如：

![core struct of log4go]({{ site.baseurl }}/attachments/2016-04-06/core_struct.png)

# 2. 特性

log4go支持类似log4j的xml配置文件，多个日志过滤器和级别设置。access日志（打印请求时间，结果，耗时等）。具体特性如下：

1. 支持多个日志Filter配置，配置文件类似log4j的xml格式，默认会自动加载当前目录下的./log4go.xml或者conf/log4go.xml。如果没有找到，则需要手动指定，或者通过字符串的形式传入Setup()函数中。

2. 当服务器重启时，如果已有log文件并且文件不满足滚动条件的时候，会尝试重用这些文件。这与Java中的通常做法一致（[alecthomas/log4go](https://github.com/alecthomas/log4go)的做法是每次重启都会生成一个新的log文件）

3. 增加`Exclude`标签，可以用于不显示指定package下的代码所打印的日志。这个特性的使用场景在于如果第三方包或者一些lib包（比如我们的数据库封装层）也使用了log4go打印日志的话，我们在最终引用他们的时候可能并不希望看到他们debug级别的日志输出。例如下面的exclude标签表示*github.com/example*和*github.com/sample*开头的包所产生的INFO及以下级别的日志都不会输出到console中。

```xml
<filter enabled="true">
  <tag>stdout</tag>
  <type>console</type>
  <level>INFO</level>
  <exclude>github.com/example</exclude>
  <exclude>github.com/sample</exclude>
</filter>
```

4. 增加access级别的日志。access日志只会打印到access.log中，同样支持滚动。access日志的打印时间是一个请求结束之后。会记录整个请求的http状态码和请求消耗的时间。

5. 对于log4go.Critical()会打印出调用堆栈。

6. 提供一个log4go.Recover()方法，仅在捕捉到panic的时候打印堆栈，否则它的行为就和log.Error()一样。使用方法：

```go
defer log4go.Recover("this is a msg: %v", "msg")
// or use with a function
defer log4go.Recover(func(err interface{}) string {
  // ... put your code here, construct the error message and return
  return fmt.Sprintf("recover..v1=%v;v2=%v;err=%v", 1, 2, err)
})
```

# 3. TODO

1. exclude需要能够在所有级别中生效
2. 移除一些无用代码

# 4. Reference

[kimiazhu/log4go](https://github.com/kimiazhu/log4go)