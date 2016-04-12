---
layout:     post
title:      "Golang Object to String Using Reflect"
date:       2016-04-11 20:05
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Golang
---

# 1. 说明
<br />

将struct对象信息方便地以人可读的方式打印到日志或者console中，go提供了一些方式，可以让我们方便地做这个事情。

# 2. fmt包标准输出
<br />

标准包`fmt`中提供了格式化字符串的支持。

```go
fmt.Printf("%v", value)
```

`%v`可以打印出你想要的方式。并且私有字段也可以通过反射获取输出值。

对于以下的`struct`：

```go
type R struct {
  L int
  w int
}
```

具体可以有三种方式进行输出：

- %v 输出只有value的格式。比如一个矩形`{100 50}`
- %+v 输出带filed name的格式，比如一个矩形`{L:100 w:50}`
- %#v 输出带type、filed name和filed value的完整形式，还是这个矩形：`main.R{L:100, w:50}`

# 3. json输出
<br />

可以调用`encoding/json`包进行格式化成json格式输出，但这个方式只会导出公开的字段，**对于unexported字段值是不会出现的**。

```go
// ToJson return the json format of the obj
// when error occur it will return empty.
// Notice: unexported field will not be marshaled
func ToJson(obj interface{}) string {
  result, err := json.Marshal(obj)
  if err != nil {
    return fmt.Sprintf("<no value with error: %v>", err)
  }
  return string(result)
}
```

# 4. 自定义的输出
<br />

如果需要更自由一些，又希望节省代码，可以使用反射的方式递归遍历一个数据结构内的每个字段和值，并根据需要进行拼接打印。

## 4.1 RefelectToString
<br />

我提供了一个ReflectToString()的实现，使用反射机制来生成一个struct的字符串表示。它与fmt包不同的地方在于，在保持通用性的同时，提供了许多配置项来控制字符串的最终格式。

具体代码附在后边，函数的签名和注释如下：

```go
  // ReflectToString return the string formatted by the given argument,
  // the number of args may be one or two
  //
  // the first argument is the print style, and it's default value is
  // StyleMedium. the second argument is the style configuration pointer.
  //
  // The long style may be a very long format like following:
  //
  //      Type{name=value}
  //
  // it's some different from fmt.Printf("%#v\n", value),
  // it's separated by comma and equal
  //
  // Then, the medium style would like:
  //
  //      {key=value}
  //
  // it's some different from fmt.Printf("%+v\n", value),
  // it's separated by comma and equal
  //
  // Otherwise the short format will only print the value but no type
  // and name information.
  //
  // since recursive call, this method would be pretty slow, so if you
  // use it to print log, may be you need to check if the log level is
  // enabled first
  // 
  // examples:
  //
  //   - ReflectToString(input)
  //   - ReflectToString(input, StringStyleLong)
  //   - ReflectToString(input, StringStyleMedium, &StringConf{SepElem:";", SepField:",", SepKeyValue:":"})
  //   - ReflectToString(input, StringStyleLong, &StringConf{SepField:","})
  func ReflectToString(obj interface{}, args ...interface{}) string 
```

我们可以通过这样一个方法来实现类似fmt.Printf()函数，并且支持一定的格式自定义：

- 短格式：只打印value，类似`fmt.Printf("%v", value)`
- 中格式：打印key和value，类似`fmt.Printf("%+v", value)`
- 长格式：打印Type、key和value，类似`fmt.Printf("%v", value)`

支持的分隔符控制有三种：

- SepElem：map、slice、array元素之间的分隔符，默认是逗号(`","`)
- SepField：每个struct属性之间的分隔符，默认是逗号+空格(`", "`)
- SepKeyValue：属性和属性值之间的分隔符，默认是等号(`"="`)

另外各种边界可定制：

- BoundaryStructStart/BoundaryStructEnd: struct边界，默认`{}`
- BoundaryMapStart/BoundaryMapEnd: map边界，默认`{}`
- BoundaryArraySliceStart/BoundaryArraySliceEnd: array和slice边界，默认`[]`
- BoundaryPointerFuncStart/BoundaryPointerFuncEnd: func边界，默认`()`
- BoundaryInterfaceStart/BoundaryInterfaceEnd: func边界，默认`()`

注意使用自定义边界时和分隔符共用同一个`Conf`配置项，对于传入的`Conf`中没有设置（也就是配置项的值为空字符串）的项目，将保留配置项的非空默认值。这就会产生一个问题，就是当我们希望把某个配置项目设置成空(`""`)的时候，需要明确地设置该值为`NONE`才行。比如我们希望我们slice输出的时候，不要带有默认的方括号`[]`,需要这样配置：

	c := &StringConf{SepElem:";", BoundaryArrayAndSliceStart:NONE, BoundaryArrayAndSliceEnd:NONE}

这样配置之后，对切片[1,2,3,4,5]将会生成以下的格式化字符串：

	1;2;3;4;5

而使用默认配置项时的输出会是这样的：

	[1,2,3,4,5]

作为对比，以下是使用标准库`fmt.Printf("%v", []int{1,2,3,4,5})`的输出：

	[1 2 3 4 5]


具体代码资源在[这里](https://github.com/kimiazhu/golib/blob/master/utils/strings.go)。

## 4.2 自定义ToString
<br />

当以上所有情况都不能满足需求时，或者对于某个输出具有非常特别的格式要求时候，可以针对每个struct内自行定义`String()`函数，在需要的时候进行调用该方法。

## 4.3 统一
<br />

我们可以定制一个方法来统一上面两种格式的输出：

```go
  // ToString return the common string format of the obj according
  // to the given arguments
  //
  // by default obj.String() will be called if this method exists.
  // otherwise we will call ReflectToString() to get it's string
  // representation
  //
  // the args please refer to the ReflectToString() function.
  func ToString(obj interface{}, args ...interface{}) string {
    if v, ok := obj.(fmt.Stringer); ok {
      return v.String()
  }
  return ReflectToString(obj, args)
}
```

这段代码实现了当存在String()方法的时候，我们调用对象的String()方法进行输出。否则使用反射构造通用的字符串格式输出。

# 5. Reference
<br />

- [strings.go](https://github.com/kimiazhu/golib/blob/master/utils/strings.go)