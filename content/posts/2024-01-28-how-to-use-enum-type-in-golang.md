---
title: "Go语言中 enum 实现方式有哪些？如何实现限制值范围？"
date: 2024-01-25T18:25:50+08:00
draft: true
comment: true
description: "枚举在编程能助我们清晰、安全地表示一组范围固定的值。以编写游戏程序为例，游戏中的角色有如战士、法师或者弓箭手，这种范围固定的值，就可以用枚举来表示。但 Go 语言中，枚举的表达方式并不像在一些其他语言中那样直接。要想在 GO 中用好枚举，需要我们了解 Go 中枚举的不同表示形式。"
---

枚举在编程能助我们清晰、安全地表示一组范围固定的值。

以编写游戏程序为例，游戏中的角色有如战士、法师或者弓箭手，这种范围固定的值，就可以用枚举来表示。但 Go 语言中，枚举的表达方式并不像在一些其他语言中那样直接。要想在 GO 中用好枚举，需要我们了解 Go 中枚举的不同表示形式。

本文将以 Go 如何使用枚举为主题，从最简单到复杂逐一介绍常见的方案。

## 使用 `iota` 和常量

在Go 中，使用 `iota` 和常量是最常见的表示枚举的方式。

什么是 `iota`？

`iota` 是 Go 中是一个非常特别的 Keyword，可以帮助按一定规则创建一系列相关的常量，与枚举的用途天然契合。

让我们来看一个例子，实现 `iota` 实现常量能力，代码如下所示：

```go
type Weekday int

const (
	Sunday    Weekday = iota // 0
	Monday                   // 1
	Tuesday                  // 2
	Wednesday                // 3
	Thursday                 // 4
	Friday                   // 5
	Saturday                 // 6
)
```

例子中，`Weekday` 类型有 7 个值，分别代表一周的七天。内部值从 0 开始，`iota` 在这里起到了自动增加的作用，让我们不用手动给每个常量赋值。`iota` 有很多骚操作，本文目标不在此，就不展开了。

这种方法的优点是简单且确实是提供了一定程度类型安全，但它也有局限性。

我觉得它主要是两点不足之处。

首先，它不能直接从字符串转换到枚举类型，以上面的代码为例，它不能从 "Sunday" 字符串转为 `Sunday` 枚举值。

其次，它的类型安全不是绝对安全，如上的 `Weekday` 类型，我们虽不能将一个类型明确的变量赋值给 `Weekday` 变量，但确定将一个非 `Weekday` 类的值赋值给它。

示例代码，如下所示：

```go
var day Weekday = 10
```

很明显，10 这个数字并非 `Weekday` 有效范围值，但却可以有效赋值而并没有报错。这对于其他机制完善的 enum 类型的语言而言，肯定是无法编译通过的。

除了最基础的实现方式，我们继续看看还有哪些其他表示形式吧。

## 支持字符串转化的枚举值

我们在开发 Web 应用时，常会遇到要将枚举值以字符串形式表示的情况，特别是在与前端交互时。那么，就让我们先尝试解决这第一个问题，如何实现 string 变量与枚举变量相互转化。

这个问题说来简单，我们可采用字符串常量作为枚举值。

示例代码，如下所示：

```go
type HttpMethod string

const (
    Get    HttpMethod = "GET"
    Post   HttpMethod = "POST"
    Put    HttpMethod = "PUT"
    Delete HttpMethod = "DELETE"
)
```

这种方法简单直观，而且也易于与 JSON 等格式进行互操作。当如果我们还想保留原始的 `iota` 的整型枚举值，是否可以实现呢？

如下定义：

```go
type HttpMethod int

const (
    Get    HttpMethod = iota
    Post
    Put
    Delete
)
```

只要在枚举类型上增加整型值与字符串两者间相互转化的方法即可。

代码如下所示：

```go

// 从 string 转为 HttpMethod
func NewFromString(method string) HttpMethod {
  switch h {
  case "Get":
    // 省略 ...
  case "Delete":
  default:
    return Get // default is Get or error, panic
  }
}

// 从 HttpMethod 转为 string
func (h HttpMethod) String() string {
  switch h {
  case Get:
    return "Get"
    // 省略 ...
  default:
    return "Unknown" // or error, panic
  }
}
```

如果去找一些开源项目，可能会发现一些实现了这种 enum 的包，你只要通过 `iota` 定义枚举类型，从字符串和枚举间转化的代码可通过命令直接生成。

robpike 开发过一个工具名为 [`stringer`](https://pkg.go.dev/golang.org/x/tools/cmd/stringer)，可直接基于类似如上 `HttpMethod` 定义生成 `String()` 方法，不过它不是完整的 enum 支持。

```go
//go:generate stringer -type=HttpMethod
type HttpMethod int

const (
    Get    HttpMethod = iota
    Post
    Put
    Delete
)
```

我们执行 `go generate` 即可为 `HttpMethod` 类型生成 `String` 方法。

但毫无疑问，我们依然存在类型安全的问题，类似 `var Hello HttpMethod = 10` 这样的代码依然有效。

继续吧！

## 结构体枚举值

GO 中其实基于结构体类型，我们也可以实现枚举效果。比如，我们创建一个颜色的枚举，不仅有颜色的名字，还有颜色的 RGB 值，当然也可以给它加上一个枚举整型值。

```go
type Color struct {
    Index int
    Name  string
    RGB   string
}

```

这样我们就有了一个颜色的枚举，每个颜色都有一个索引、名字和 RGB 值。

如何使用呢？

类似于前面的方式，我们直接定义，如下所示：

```go
var (
	  Red   = Color{0, "Red", "#FF0000"}
	  Green = Color{1, "Green", "#00FF00"}
	  Blue  = Color{2, "Blue", "#0000FF"}
)
```

这种方法比较灵活，但也相对复杂。

当然，好处肯定是有的，现在能存储的信息也更加丰富，前面类似于整型与字符串的各种转化都变的轻而易举了。整型数值 `Color.Index`、字符串 `Color.Name`。

## 结构体类似命名空间效果

如果我们有很多枚举类型，担心可能会出现命名冲突。这时，可以使用结构体来创建一个“命名空间”，把相关的枚举值组织在一起：

示例代码如下所示：

```go
var Colors = struct {
    Red, Green, Blue Color
}{
	  Red   = Color{0, "Red", "#FF0000"}
	  Green = Color{1, "Green", "#00FF00"}
	  Blue  = Color{2, "Blue", "#0000FF"}
}
```

上面的例子中定义了 `Colors` 变量，它是匿名结构体类型，字段名表示颜色，我们可通过 `Colors.xxx` 形式调用颜色。

这个实现方式看起来似乎很不优雅，也很鸡肋，其实我完全可以用新建 `package` 实现。不过既然发现了这个方案，就

## 类型安全？

到这里，其实所有实现方式都没有解决一个问题，那就是定义完枚举后，依然继续添加新的枚举值。简单来说，类型安全。

如果我真的想实现这样的能力呢？

例如，我想防止直接通过类似 `HttpMethod(1)` 进行枚举，我的思考是，可尝试考虑将枚举实现封装成一个 package，将类型小写，如 `httpMethod`，暴露它的类似 `FromString` 和其它函数强制使用转化函数。

```go
package httpmethod

type httpmethod string

const (
  Get  = "Get"
  Post = "Post"
)

func FromString(method string) httpmethod {
  switch method {
  case "Get":
    return Get
  case "Post":
    return Post
  }
}
```

但我想说，方法可能挺好，但确实有点多余。很多情况，我们也不用考虑这个问题。毕竟，我也不会随意创建新的枚举值。

## 真实场景

我对真实场景下枚举使用的考虑，有价值之处主要在与其他系统的对接。

举例而言，如来自前端 API 或数据库，有时可能出现一些异常值。对这类场景，通过前面介绍，可提供转化函数，在其中设置检查规则。如果发现异常选择丢弃，执行如 error 或 panic。

而对于内部系统，如果使用类似于 `protobuffer` 协议，可在协议上限定好枚举范围。当然，也可能因为发布时间次序或者兄弟团队忘记通知等，导致系统间枚举值对不齐的情况，也会按上面的逻辑丢弃、error 等，便于即使发现。

其实，对于团队合作这类场景，最好的解决方式，还是要在设计系统时，考虑上下游的兼容性，而不是每当有变动，全员乱糟糟，这最容易导致生产事故了。

其实无论哪一种情况，重点在于保证进入系统的数据是否可通过转化检测，而不是多此一举，限制类似于 `HttpMethod("Get")` 的类型转化，因为没有人会这么写代码。

## 总结

Go 语言中，枚举的表达方式多种多样。从简单的 `iota` 到复杂的结构体方式，每种方法都有其适用场景。我们作为开发者，最好是根据自己的具体需求，选择合适的实现方式。

最后，希望这篇文章能帮助你在使用 Go 语言时，更加灵活且游刃有余地使用枚举 enum。

