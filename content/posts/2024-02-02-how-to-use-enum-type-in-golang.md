---
title: "Go语言中 enum 实现方式有哪些？一定要绝对类型安全吗？"
date: 2024-02-02T08:00:00+08:00
draft: false
comment: true
description: "Go 语言中，枚举的表达方式并不像在一些其他语言中那样直接。要想在 GO 中用好枚举，需要我们了解 Go 中枚举的不同表示形式。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 技巧第十二篇，系列文章查看：[Go 语言技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

枚举，即 enum，可用于表示一组范围固定的值，它能助我们写出清晰、安全的代码。

以编写游戏程序为一个简单案例：游戏中的角色有如战士、法师或者弓箭手，这种范围固定的值，就可以用枚举来表示。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-08.png)

但 Go 中，枚举的表现方式不像在某些其他语言中那样直接。我们要想在 Go 中用好枚举，就要了解 Go 中枚举的不同表示形式和使用注意点。

本文将以 Go 语言中如何使用枚举为主题，从最简单到复杂逐一介绍常见的方案。

## 使用 `iota` 和常量

在 Go 中，使用 `iota` 和常量是最常见的表示枚举的方式。

什么是 `iota`？

`iota` 是 Go 中是一个非常特别的 Keyword，它可以帮助我们按一定规则创建一系列相关的常量，而无需手动为每个变量单独赋值。这一点与枚举的用途天然契合。

不了解上面文字的含义？

让我们来看一个例子，基于 `iota` 快速创建特定规则的常量。

示例代码，如下所示：

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

例子中，`Weekday` 类型有 7 个值，分别代表一周的七天。内部值从 0 开始，`iota` 自动增加赋值给每个常量，从 Sunday 到 Saturday 分别赋值 0-7。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-02.png)

现在，我们就不用手动给每个常量赋值。

`iota` 还有很多骚操作，本文目标不在此，就不展开了。

这种方法的优点是简单，提供了一定程度上类型安全，但它也有局限性。

我觉得主要是两点不足。

首先，对比其他语言的枚举，它不能直接从字符串转换到枚举类型，以上面代码为例，它不能从 "Sunday" 字符串转为 `Sunday` 枚举值。

其次，它的类型安全不是绝对安全。

如上的 `Weekday` 类型，我们虽不能将一个明确类型的变量赋值给 `Weekday` 类型变量：

```go
day := 0 // int
// compiler: cannot use day (variable of type int) 
// as Weekday value in variable declaration [IncompatibleAssign]
var sunday Weekday = day 
```

但却可以将一个非 `Weekday` 类的字面量赋值给它。

```go
// 字面量 10 赋值给类型为 Weekday 的 day 变量
var day Weekday = 10 
```

很明显，10 这个数字并不在 `Weekday` 的有效范围，但却可以有效赋值而并没有报错。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-03.png)

如果是其他枚举机制完善的 enum 类型的语言，肯定是无法编译通过的。

除了最基础的实现方式，我们继续看看还有哪些其他表示形式吧。

## 支持字符串转化的枚举值

我们在开发 Web 应用时，常会遇到要将枚举值以字符串形式表示的需求，特别是在与前端对接时。那么，就让我们先尝试实现这一个需求，string 变量与枚举变量相互转化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-04.png)

这个问题说来简单，Go 语言中，我们可采用字符串常量作为枚举值。

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

这种方法简单直观，而且也易于与 JSON 等数据格式转换。

```go
type Request struct {
	Method HttpMethod
	URL    string
}

func main() {
	r := Request{Method: Get, URL: "https://zhihu.com"}
	result, _ := json.Marshal(r)
	fmt.Printf("%s\n", result)
}
```

输出：

```bash
{"Method":"GET","URL":"https://zhihu.com"}
```

当如果我们还想保留原始的 `iota` 的整型枚举值，毕竟它更轻量，占用内存空少。这是否可以实现呢？我们尝试一下吧。

定义如下：

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

我们实现从 string 构造 enum 方法，和从 enum 类型转为 string 的 String 方法。

这里存在的一个问题，如果希望支持友好的 JSON 序列化反序列化的话，即枚举值使用字符串形式表示，则需要为 `HttpMethod` 新增方法，实现 json.Marshaler 和 json.Unmarshaler 接口，自定义转化过程。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-05.png)

代码如下：

```go
// MarshalJSON 实现 json.Marshaler 接口
func (h HttpMethod) MarshalJSON() ([]byte, error) {
    return json.Marshal(h.String())
}

// UnmarshalJSON 实现 json.Unmarshaler 接口
func (h *HttpMethod) UnmarshalJSON(data []byte) error {
    var method string
    if err := json.Unmarshal(data, &method); err != nil {
        return err
    }
    *h = NewFromString(method)
    return nil
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

```
go generate
```

这里有个提前，要单独安装下 `stringer` 命令。

不过，即使到现在，依然存在类型安全的问题，类似 `var Hello HttpMethod = 10` 这样的代码依然有效。

继续吧！

## 结构体枚举值

GO 中可基于结构体类型，实现枚举效果。

举例说明，我们创建一个颜色的枚举，要求不仅有颜色的名字，还有颜色的 RGB 值。同时，为了方便记录，我们可以给它加上一个枚举整型值。

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

好处也比较明显，如现在能存储的信息也更加丰富，前面类似于整型与字符串的各种转化都变的轻而易举了。我们直接整型数值 `Color.Index`、字符串 `Color.Name`。

不过，如果要最大化与其他库结合，如自定义 JSON 转化规则，要实现 JSON 序列和反序列接口，字符串格式化要实现 `Stringer` 接口等。

还有，这种结构其实不是常量类型的，就存在数据可更改的问题。不过，有这个安全需求的话，可考虑将成员字段私有化，通过方法变更即可。

## 结构体类似命名空间效果

在网上看到个有点傻的设计，顺便也提一下吧。

假设，我们有很多枚举类型，担心可能会出现命名冲突，可以用结构体来创建一个“命名空间”，把相关的枚举值组织在一起：

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

我初期看到这个写法，还以为限定了类型可定义的枚举值范围。发现其实不是，我依然可使用 `Color` 类型定义新值。

这很不优雅，也很鸡肋，其实我完全可以新建 `package` 实现。不过既然发现了这个方案，就写到这里吧。

## 类型安全？

到这里，其实所有实现方式都没有解决一个问题，那就是定义完枚举后，依然继续添加新的枚举值。

我真的想实现这样的能力呢？该如何做呢？

以前面 `HttpMethod` 为例，我要做的就是禁止通过 `HttpMethod(1)` 创建新枚举值。

这不是很简单吗？

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-07.png)

我们只要将枚举实现封装成一个 package，将类型小写，如 `httpMethod`，暴露它的类似 `FromString` 和其它函数，实现强制通过转化函数它。

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

现在，枚举创建必须通过方法，我们就可以在其中实现限定创建规则。

方法可能挺好，但好像没见到这么玩的？

为什么呢？

我的猜想是，开发时我们不会随意创建新的枚举值，对于边界数据的传递，确保通过转化函数处理不就行了吗？

## 真实场景

对真实场景下枚举的使用，有价值之处主要在与其他系统的对接。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-06.png)

举例而言，如来自前端 API 或数据库，有时可能出现一些异常值。对这类场景，通过前面介绍，可提供转化函数，在其中设置检查规则。如果发现异常选择丢弃，执行如 error 或 panic。

而对于内部系统，如果使用类似于 `protobuffer` 协议，可在协议上限定好枚举范围，标记异常数据等。

当然，可能出现因为发布时间次序或者兄弟团队忘记通知等，导致系统间枚举值对不齐的情况，也会按上面的逻辑丢弃、error 等，便于即使发现。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-09.png)

对于团队合作这类场景，最好的解决方式，还是要在设计系统时，考虑上下游的兼容性，而不是每当有变动，全员乱糟糟，这最容易导致生产事故了。

其实无论哪一种情况，重点在于保证进入系统的数据是否可通过转化检测，而不是多此一举，限制类似于 `HttpMethod("Get")` 的类型转化，因为没有人会这么写代码。

## 总结

Go 语言中，枚举的表达方式多种多样。从简单的 `iota` 到复杂的结构体方式，每种方法都有其适用场景。作为开发者，最好是根据自己的具体需求，选择合适的实现方式。

最后，希望这篇文章能帮助你在使用 Go 语言时，更加灵活且游刃有余地使用枚举 enum。

博客地址：[Go语言中 enum 实现方式有哪些？一定要类型安全吗？](https://www.poloxue.com/posts/2024-02-02-how-to-use-enums-type-in-golang/)
