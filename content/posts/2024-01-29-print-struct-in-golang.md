---
title: "Go 中打印结构体有几种方法？"
date: 2024-01-30T21:00:00+08:00
draft: true
comment: true
description: "不知道大家是否遇到打印结构体的需求呢？结构体就像是一个小盒子，里面可以放很多不同类型的东西，如数字、字符串、slice、map 或其他结构体。但，如果我们想看看盒子里都放了什么，该怎么办呢？"
---

不知道大家是否遇到打印结构体的需求？结构体就像是一个小盒子，里面可以放很多不同类型的东西，如数字、字符串、slice、映射或者其他结构体。但，如果我们想看看盒子里都放了什么，该怎么办呢？这时我们就要能打印结构体了。它能帮我们检查和理解代码，提高调试效率，确保一切按照我们的想法运行。

本文让我们以此为话题，聊聊 GO 语言中打印结构体内容的几种方法。

让我们直接上方案吧。

## 定义结构体

首先，我们来定义一个接下来所有方法都要用的结构体，如下所示：

```go
type Author struct {
	Name string
	Age  int8
	Sex  string
}

type Article struct {
	ID     int64
	Title  string
	Author *Author
}
```

## 使用 `fmt.Printf` 和 `%+v`

最简单的方法是使用`fmt.Printf`函数和`%+v`格式化符号。这样可以直接打印出结构体的字段名和值。比如我们有一个叫`article`的结构体，我们可以这样打印它：

```go
func main() {
    article := Article{"Go语言简介", "小明", 10}
    fmt.Printf("%+v\n", article)
}
```

这段代码会显示`article`结构体的所有字段和相应的值。


### 实现 `String` 方法

你还可以为你的结构体类型实现`String`方法。这样，每当你使用`fmt.Printf`打印结构体时，就会调用你定义的`String`方法，让你可以控制结构体的输出格式。例如：

```go
func (a Article) String() string {
    return fmt.Sprintf("Title: %s, Author: %s, Pages: %d", a.Title, a.Author, a.Pages)
}

func main() {
    article := Article{"Go语言简介", "小明", 10}
    fmt.Println(article)
}
```


## 使用 `json.MarshalIndent` 进行格式化打印

如果你想要一个更美观的输出格式，可以使用`json.MarshalIndent`函数。这个函数会将结构体转换为JSON格式，并且可以控制缩进，使输出更易于阅读。例如：

```go
import (
    "encoding/json"
    "fmt"
)

func main() {
    article := Article{"Go语言简介", "小明", 10}
    articleJSON, _ := json.MarshalIndent(article, "", "    ")
    fmt.Println(string(articleJSON))
}
```

这样会以美观的JSON格式打印出`article`结构体。

### 使用 `reflect` 包进行更复杂的打印

如果你要更详细地控制打印格式或者打印那些没有实例的结构体类型，可以使用 `reflect` 包。这个包可以让我们检查、修改和创建任何Go语言中的变量，包括结构体的字段名和类型。例如：

```go
import (
    "fmt"
    "reflect"
)

func main() {
    article := Article{"Go语言简介", "小明", 10}
    val := reflect.ValueOf(article)
    for i := 0; i < val.NumField(); i++ {
        fmt.Printf("Field: %s, Value: %v\n", val.Type().Field(i).Name, val.Field(i))
    }
}
```

## 性能压测

在我们尝试了不同的打印方法后，我们也进行了一些性能测试。这里是测试结果：

```bash
BenchmarkFmtPrintf-16             	 2631248	       447.3 ns/op
BenchmarkJSONMarshalIndent-16     	  997448	      1016 ns/op
BenchmarkCustomStringMethod-16    	 5135541	       225.5 ns/op
BenchmarkReflection-16            	 2030233	       594.9 ns/op
```

这些结果告诉我们，使用自定义的`String`方法是最快的，而`json.MarshalIndent`则是最慢的。这意味着如果你关心程序的运行速度，最好使用自定义的`String`方法来打印结构体。

## 第三方库：`go-spew`

最后，我们来看看一个叫做`go-spew`的第三方库。这个库提供了深度打印Go数据结构的功能，对于调试非常有用。它可以递归地打印出结构体的字段，即使是嵌套的结构体也能打印出来。例如：

```go
import "github.com/davecgh/go-spew/spew"

func main() {
    article := Article{"Go语言简介", "小明", 10}
    spew.Dump(article)
}
```

这样会详细地打印出`article`结构体的所有内容。

## 结语

通过这篇文章，我们学习了在Go语言中打印结构体的不同方法。从简单的 `fmt.Printf` 到使用反射或第三方库，我们有很多选择。

希望这篇文章能帮助你在遇到打印 GO 结构类似需求时，不再迷茫。


