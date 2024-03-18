---
title: "Go 语言实现可选参数：重载？变长参数？"
date: 2024-01-27T08:00:00+08:00
draft: false
comment: true
description: "在编程时，常会遇到这样的情况：一个函数偶尔需要一些不固定的选项参数。一些语言中，通过重载或者支持可选参数解决这个问题。但在 Go 中，情况有所不同，因为 Go 不支持函数重载，也没有内置可选参数功能。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 小技巧第八篇，系列文章查看：[Go 语言小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

我们编程时，常会遇到：一个函数在大多数情况下只需要几个参数，但偶尔也需要一些不固定的选项参数。在一些语言中，通过重载或者可选参数来解决这个问题。但 Go 中，情况有所不同，因为 Go 不支持函数重载，也没有内置可选参数功能。如果就想要这样的能力，如何在 Go 中实现？

本文将基于这个主题展开，一步步介绍 GO 中实现可选参数的几种方法。

## 方法1：可变长参数（Variadic Args）

GO 不支持可选参数，但它好在还是支持可变长参数，即允许函数接受任意数量的参数。这是通过在参数类型前加上 `...` 来实现的。

示例代码，如下所示：

```go
func printNumbers(numbers ...int) {
    for _, number := range numbers {
        fmt.Println(number)
    }
}

func main() {
    printNumbers(1, 2)
    printNumbers(1, 2, 3, 4)
}
```

在上面的例子中，我们定义了一个 `printNumbers` 函数，它可以接受任意数量的整数作为参数。

这种方法主要还是适合于所有参数都是同一类型的情况。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-02.png)

但如果参数类型不同怎么办呢？

当然，一种方式是，通过使用 `...interface{}` 继续基于可变长参数实现，但这毫无疑问会增加反射或者类型选择或推导的开销，同时每个位置的参数按索引确定，代码复杂度必然提高，可读性会大大降低，

那么，是否还有更好的方式呢？

## 方法2：使用Map

当你需要传递不确定数量且类型不同的参数时，可以使用 `map` 实现。

```go
func setConfig(configs map[string]interface{}) {
    if val, ok := configs["timeout"]; ok {
        fmt.Println("Timeout:", val)
    }
    if val, ok := configs["path"]; ok {
        fmt.Println("Path:", val)
    }
}

func main() {
    setConfig(map[string]interface{}{
        "timeout": 30,
        "path":    "/usr/bin",
    })
}
```

在这个例子中，`setConfig` 函数接受一个 `map` 作为参数，其中键是配置项的名称，值是配置项的值。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-03.png)

这种方法的缺点是失去了类型安全性，也需要在运行时对 `interface{}` 类型参数进行类型断言，只是相对于变长参数的方式，类型相对比较明确。

有没有不会失去类型安全的方法呢？

## 方法3：使用结构体（Structs）

如果我们想要类型安全，同时又想要可选参数的灵活性，结构体似乎是一个不错的选择。但每次调用函数时都需要创建一个新的结构体实例，这会不会太麻烦？

```go
type Config struct {
    Timeout int
    Path    string
}

func setConfig(config Config) {
    fmt.Println("Timeout:", config.Timeout)
    fmt.Println("Path:", config.Path)
}

func main() {
    setConfig(Config{
        Timeout: 30,
        Path:    "/usr/bin",
    })
}
```

这种方法的好处是类型安全，并且可以清晰地看到哪些参数被设置了。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-04.png)

缺点是每次调用函数时都需要创建一个新的结构体实例。

## 方法4：函数选项模式（Functional Options Pattern）

那么，有没有一种方法既能保持类型安全，又能提供灵活的可选参数呢？函数选项模式似乎提供了这样的可能。

```go
type Config struct {
    Timeout int
    Path    string
}

type Option func(*Config)

func WithTimeout(timeout int) Option {
    return func(c *Config) {
        c.Timeout = timeout
    }
}

func WithPath(path string) Option {
    return func(c *Config) {
        c.Path = path
    }
}

func NewConfig(opts ...Option) *Config {
    config := &Config{}
    for _, opt := range opts {
        opt(config)
    }
    return config
}

func main() {
    config := NewConfig(
        WithTimeout(30),
        WithPath("/usr/bin"),
    )
    fmt.Println(config)
}
```

在这个例子中，我们定义了`Config` 结构体和 `Option` 类型，`Option` 是一个函数，它接受一个`*Config`参数。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-05-v2.png)

我们还定义了`WithTimeout`和`WithPath`函数，它们返回一个`Option`。这样，我们就可以在调用`NewConfig`函数时，通过传递不同的选项修改 `Config` 结构中的字段，构建不同的配置。

这种方法的好处是非常灵活，并且可以在不破坏现有代码的情况下扩展 API。缺点是实现起来比较复杂，可能需要一些时间来理解。

## 总结

这篇博文介绍了在Go语言中实现可选参数的几种方法：可变长参数、使用Map、结构体和函数选项模式。每种方法都有其适用场景和优缺点，你可以根据自己的需要选择合适的方法。

博文地址：[Go 语言实现可选参数：重载？变长参数？](https://www.poloxue.com/posts/2024-01-25-optional-parameters-in-golang/)
