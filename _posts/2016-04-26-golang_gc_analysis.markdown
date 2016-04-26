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

# 1. GC日志格式

```log
gc # @#s #%: #+...+# ms clock, #+...+# ms cpu, #->#-># MB, # MB goal, # P
```

- `gc #` GC进行次数，每次GC自增
- `@#s` 自程序启动以来的时间
- `#%` 自程序启动以来，花在GC上的时间的百分比
- `#+...+#` 这个有两段，一段是wall clock时间，一段是cpu时间。每段内部都是用+号分隔
	- wall clock时间段分三部分：STW清理终止+扫描、同步、标记终止(_GCmarktermination)+STW结束阶段
	- cpu time时间段也分三部分：broken in to assist time+后台GC时间+空闲GC时间
- `#->#-># MB` 三个阶段的堆大小，三个阶段分别是：GC开始时堆大小->GC结束时堆大小->实时堆大小
- `# MB goal` 目标堆大小
- `# P` 处理器使用数量

# 2. Golang循环引用测试

```go:
// Description: src
// Author: ZHU HAIHUA
// Since: 2016-04-26 09:46
package main

import (
	"runtime"
	"fmt"
	"strconv"
)

type Person struct {
	name string

	data []byte
	Apart *Apartment
}

type Apartment struct {
	addr string
	data []byte

	Tenants *Person
}

func main() {
	fmt.Println("start test gc")

	for i := 0; i <= 50000; i++ {
		apartment := &Apartment{addr: "Zhuhai, CN: " + strconv.Itoa(i), data: make([]byte, 1 << 16)}
		tenant := &Person{name: "HAIHUA ZHU: " + strconv.Itoa(i), data: make([]byte, 1 << 16)}
		runtime.SetFinalizer(apartment, func(a *Apartment) {
			//fmt.Printf("Apartment in [%s] removed\n", a.addr)
		})
		runtime.SetFinalizer(tenant, func(p *Person) {
			//fmt.Printf("Tenant [%s] removed\n", p.name)
		})

		(*apartment).Tenants = tenant
		(*tenant).Apart = apartment

		tenant = nil
		apartment = nil
	}

	fmt.Println("end test gc")
}
```

输出如下（根据环境不同，具体数值也会有不同）：

	// ... other log ...
	gc 10 @0.064s 15%: 0+2.5+1.0 ms clock, 0+0/2.0/5.5+4.0 ms cpu, 1185->1188->1189 MB, 1216 MB goal, 4 P
	gc 11 @0.103s 11%: 0.50+6.5+0.50 ms clock, 2.0+0/5.0/15+2.0 ms cpu, 2313->2332->2333 MB, 2372 MB goal, 4 P
	gc 12 @0.205s 7%: 0+18+0.50 ms clock, 0+0/18/40+2.0 ms cpu, 4512->4533->4534 MB, 4628 MB goal, 4 P
	gc 13 @0.287s 18%: 0.50+0+38 ms clock, 2.0+0/18/40+152 ms cpu, 6260->6260->6262 MB, 6260 MB goal, 4 P (forced)
	end test gc

当注释掉40行时，去掉了循环引用，此时可以可以发现堆大小明显下降。

	// ... other log ...
	gc 22 @0.470s 3%: 0+3.0+0.50 ms clock, 0+0/2.5/4.0+2.0 ms cpu, 1220->1231->828 MB, 1251 MB goal, 4 P
	gc 23 @0.589s 2%: 0+3.5+0.50 ms clock, 0+0/2.5/5.0+2.0 ms cpu, 1594->1607->1080 MB, 1635 MB goal, 4 P
	gc 24 @0.758s 2%: 0+6.0+0.50 ms clock, 0+0/5.5/5.5+2.0 ms cpu, 2081->2103->1419 MB, 2134 MB goal, 4 P
	gc 25 @0.984s 2%: 0+8.5+0.50 ms clock, 0+0/6.5/13+2.0 ms cpu, 2724->2724->1823 MB, 2794 MB goal, 4 P
	end test gc

如果打开line33或者line36行的注释，还可以看到每次系统执行gc时回收了哪些对象。可以发现，有循环引用的时候，GC并没有释放内存。

值得注意的是，当循环引用不是通过指针，而是对象来引用的话，是无法通过编译的，编译器会报递归引用错误终止执行。

# 3. Reference 

1. [GC算法的概要](http://gad.qq.com/article/detail/7150922)
2. [Go 1.4+ Garbage Collection (GC) Plan and Roadmap](https://docs.google.com/document/d/16Y4IsnNRCN43Mx0NZc5YXZLovrHvvLhK_h0KN8woTO4/edit#heading=h.o8eay7ieosat)
3. 