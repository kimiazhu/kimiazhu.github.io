---
layout:     post
title:      "Golang FAQ"
date:       2016-02-29 12:05
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Golang
    - Github
---

# 目录：
{:.no_toc}
* any list
{:toc}

# 1. Windows下拉取GitHub私有仓库

mac或者linux下可以很容易地配置ssh密钥，而在windows下可以用两种方式拉取私有仓库。

## 1.1) 生成github access token：

进入github -> settings -> Personal access token, 生成一个带repo权限的token。


## 1.2) 配置git

使用如下配置命令：

	$> git config --global url."https://${GITHUB_TOKEN}:x-oauth-basic@github.com/".insteadOf "https://github.com/"

其中${GITHUB_TOKEN}换成自己刚刚生成的token值

## 1.3) 拉取代码

	$> go get [-u] github.com/kimiazhu/private_repo

# 2. 结构体嵌套定义和初始化

Go的结构体嵌套定义和匿名结构体初始化，有时候在一次性时候结构体时会比较方便。当然，这种方式比较晦涩，大量使用并没有任何好处。

## 2.1）嵌套定义

结构体可以嵌套定义，内部还可以定义数组，嵌套定义的结构体是匿名的，同时也可以指定tag。

```go
type A struct {
  VA string `tag:"va"`
  B  struct { // B是一个内嵌结构体
    VB string `tag:"vb"`
  } `tag:"b"`
  C []struct { // C是一个结构体数组
    VC string `tag:"vc"`
  } `tag:"c"`
}
```

## 2.2）初始化

嵌套定义的结构体是匿名的，初始化的时候仍要将结构体重写一遍。需要注意的是，匿名结构体内属性的tag也要重写，并且和之前定义的要写成一样，否则会报类型不匹配的错误。

```go
A := A {
  VA: "valueA",
  B: struct {
    VB string `tag:"vb"`
  } {VB: "valueB"},
  C: []struct {
    VC string `tag:"vc"`
  } { {VC: "valueC1"}, {VC: "valueC2"} },
}
```

# 3. 复制流

复制流的需求出自于使用gin中间件处理请求中的签名或者时间错等与业务关系不大的逻辑时，需要从request.Body中读取这些值，但是又不能影响原有的request.Body，因为后面正常业务处理的时候还需要从中读取值。

## 3.1）Method1:

先做一个简单的reader用于从buffer中读取数据，然后用ioutil.ReadAll()读取request.Body暂存到buffer中，在用完requst.Body之后，再将reader重新指向我们做的reader

```go
type reader struct {
  *bytes.Buffer
}

func (r reader) Close() error {
  return nil
}

// copy stream to buffer
buf, _ := ioutil.ReadAll(c.Request.Body)
// Body has been consumed, put it back
c.Request.Body = reader{bytes.NewBuffer(buf)}
// continue to using request...
if c.BindJSON(&m) { // this will comsume the request.Body again
  // ok, read it back, again
  c.Request.Body = reader{bytes.NewBuffer(buf)}
}
```

## 3.2）Method2:

参考google httputil包中DumpRequest()方法的实现。

```go
// One of the copies, say from b to r2, could be avoided by using a more
// elaborate trick where the other copy is made during Request/Response.Write.
// This would complicate things too much, given that these functions are for
// debugging only.
func drainBody(b io.ReadCloser) (r1, r2 io.ReadCloser, err error) {
  var buf bytes.Buffer
  if _, err = buf.ReadFrom(b); err != nil {
    return nil, nil, err
  }
  if err = b.Close(); err != nil {
    return nil, nil, err
  }
  return ioutil.NopCloser(&buf), ioutil.NopCloser(bytes.NewReader(buf.Bytes())), nil
}

func DumpBodyAsReader(req *http.Request) (reader io.ReadCloser, err error) {
  if req == nil || req.Body == nil {
    return nil, errors.New("request or body is nil")
  } else {
    reader, req.Body, err = drainBody(req.Body)
  }
  return
}

func DumpBodyAsBytes(req *http.Request) (copy []byte, err error) {
  var reader io.ReadCloser
  reader, err = DumpBodyAsReader(req)
  copy, err = ioutil.ReadAll(reader)
  return
}
```

# 4. Goroutine Panic

任何一个Goroutine产生的panic如果未经任何recover到达栈顶会导致整个服务器进行崩溃。（并且这是Google故意设计而为之的）那么在生产环境如何避免这种莫名的崩溃，或者在崩溃时候记录下日志就是个重要的环节。

## 4.1 防止崩溃

每个go调用，都执行recover尝试恢复。可以自己手写，我[在这里](https://github.com/kimiazhu/golib/tree/master/safego "safego")也做了一个封装。

```go
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    sofego.Go(func() {
      panic("OMG!")
    })
  })
  http.ListenAndServe(":8080", nil)
```

默认会记录日志和堆栈信息写入到os.Stderr。如果你需要增加回调处理自己的业务，可以增加第二个参数：

```go
  var MyHandler = func(err interface{}) {
    log4go.Critical("panic recovered: %v", err)
  }
  
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    sofego.Go(func() {
      panic("OMG!")
    }, MyHandler)
  })
  http.ListenAndServe(":8080", nil)
```

## 4.2 崩溃补救

一旦服务器崩溃，首要的是保留日志，给责任人发送通知，尝试恢复服务。这里只关注现场保留，即记录崩溃日志。

网上有很多将os.Stderr重定向到文件的例子，也有[使用syscall.dup2进行dump记录的](http://www.cnblogs.com/ghj1976/p/4276390.html)。

这里我实践的方式不改动任何代码，直接在Linux部署应用时将系统标准错误重定向到一个文件：

	$> nohup ./server > /dev/null 2>stderr.log &