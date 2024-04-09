---
title: "我想用 Go 的 plugin 机制实现热更新，我失败了"
date: 2024-04-09T15:12:11+08:00
draft: false
comment: true
description: "昨天发了一篇名为 'entr 一个通用的热重启方案' 的文章，写完这个命令的简单使用后，我开始思考一个问题：如 Go 这样的静态编译型语言是否能实现热更新？如果能，该如何实现呢？"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-09-try-hot-update-using-plugin-package-in-golang-01.png)

昨天发了一篇名为 "entr 一个通用的热重启方案" 的文章，写完这个命令的简单使用后，我开始思考一个问题：Go 这样的静态编译型语言是否能实现热更新？如果能，该如何实现呢？

## 什么是热更新？

先简单说下什么是热更新。

热更新，或称热重载或动态更新，是一种软件更新技术，允许程序在运行时，不停机更新代码或资源。这种技术特别适用于需要高可用性的场景，如线上服务和游戏等，从而减少或消除因更新而造成的服务中断时间。

热更新有不同场景，常见的如：

**代码热替换**

动态替换或更新应用程序中的一部分代码。这通常需要特定的编程语言支持或运行时支持，如 Java 的类加载机制或 Go 的插件系统（其实无法实现）。

**资源热更新**

在不更改任何执行代码的情况下，更新应用程序使用的资源文件，如配置文件、图像或其他媒体资源。

**状态热迁移**

在更新过程中，将应用程序的状态从旧版本迁移到新版本，确保数据的连续性和一致性，如要考虑登录态、连接状态、执行中的事务等等。

简单归纳，这三种场景分别主要作用于代码层、资源层和逻辑层。而不同的场景有不同的方案，而后两者具有语言无关性。

## 实现方案

本文将主要关心的是第一种场景，即与编程语言相关的方案。具体描述为，如何在 Go 中动态替换或者说更新应用中的一部分代码。

Go 语言（通常被称为 Golang）在设计上是一种静态、编译型的语言。这意味着 Go 程序在运行前要被编译成机器代码。相比动态语言，静态编译型语言在实现热更新方面面临更多挑战。不过还是想尝试下 Go 能否可以实现热更新。

我们上面提到 Go 中实现这个代码层面的热更新能力，要借助于一个叫 plugin 系统的技术，我在网上搜索了半天，也是这个方案。不过我提前打个预防针，我的测试告诉我，Go 的插件机制其实不支持这个能力。

- go 的 plugin 机制是从 go1.8 引入，是一个实验特性。
- 支持的是系统是类 Unix 系统（Linux 和 MacOS），不支持 win。
- 只能加载不能卸载，且加载内容无法修改。

主要是最后一点，不支持 plugin 库的重载和卸载，我们就无法用它实现热更新了。Go 本身是基于静态库编译，这是它的优势，易于分享部署和发布。而这个 plugin 动态库机制，就只有动态库节省内存这个不是优势的优势。

不仅感慨，怪不得看到不少评论说 Go 的插件机器很鸡肋。

如果你关心验证过程，可继续源码实现部分。

## 开始验证

Go 1.8 引入的这个的插件系统（`plugin` 包），允许 Go 程序动态地加载其他编译好的 Go 代码作为插件。这个机制可以用来实现某种形式的热更新：

如何实现呢？

假设，我们要实现一个名为 greetings.so 的插件，源码文件是 `greetings.go`，部分源码如下所示： 

```go
//export Greet
func Greet(name string) {
    fmt.Println("Hello,", name, "from the plugin!")
}
```

为了将其编译为一个插件，我们要使用 `-buildmode=plugin` 选项编译。

```bash
$ go build -o greetings.so -buildmode=plugin greetings.go
```

在程序中加载这个插件，核心代码如下所示：

```go
func main() {
  // 加载插件
  plug, err := plugin.Open("greetings.so")
  if err != nil {
  	log.Fatal(err)
  }

  // 查找插件中的Greet符号
  symGreet, err := plug.Lookup("Greet")
  if err != nil {
  	log.Fatal(err)
  }

  // 断言Greet的类型
  var greetFunc func(string)
  greetFunc, ok := symGreet.(func(string))
  if !ok {
  	log.Fatal("Plugin has no 'Greet(string)' function")
  }

  // 使用字符串参数调用Greet函数
  greetFunc("World")
}
```

运行程序，输出如下：

```bash
$ go run main.go
Hello, World from the plugin!
```

是我们预期的结果。

## 尝试热更新

既然，我们能在主程序动态加载 `.so` 文件，那是不是就能通过检查 `.so` 文件的状态，确定是否要重新加载这个代码片段呢？

基本思路：加载 `.so` 文件时，记录其更新时间，在每次调用它实现的函数时，检查当前 `.so` 文件的更新时间，如果大于最新加载时间，重新加载执行即可。

我们可以定义个结构体，管理在 `greetings.so` 中的所有函数。

```go
// Greetings 管理greetings插件的加载和调用
type Greetings struct {
  Path        string        // 插件文件路径
  lastModTime time.Time     // 插件最后更新时间

  greetFunc   func(string)  // Greet 函数引用
}

// NewGreetings 创建并返回一个新的 Greetings 实例
func NewGreetings(pluginPath string) *Greetings {
  return &Greetings{Path: pluginPath}
}
```

实现一个内部方法，在调用 `.so` 文件中的函数时，检查插件库的更新状态，如果发现当前的库更新时间大于之前加载时的更新时间，重新加载。

```go
// tryLoadPlugin 尝试加载或重新加载插件
func (g *Greetings) tryLoadPlugin() {
  info, err := os.Stat(g.Path)
  if err != nil {
    log.Fatal("Failed to stat plugin file:", err)
  }

  modTime := info.ModTime()
  // 如果插件文件有更新，则重新加载插件
  if modTime.After(g.lastModTime) {
    log.Println("Detected plugin update, reloading...")
    g.lastModTime = modTime

    plug, err := plugin.Open(g.Path)
    if err != nil {
      log.Fatal("Failed to open plugin:", err)
    }

    symGreet, err := plug.Lookup("Greet")
    if err != nil {
      log.Fatal("Failed to find Greet symbol:", err)
    }

    var ok bool
    g.greetFunc, ok = symGreet.(func(string))
    if !ok {
      log.Fatal("Plugin has no 'Greet(string)' function")
    }
  }
}
```

现在，将 `Greet` 添加为 `Greetings` 结构体的方法即可，实现起来非常简单，如下所示：

```go
// Greet 调用插件中的 Greet 函数
func (g *Greetings) Greet(name string) {
    g.tryLoadPlugin() // 首次运行或插件更新后，尝试加载插件

    if g.greetFunc != nil {
        g.greetFunc(name) // 调用插件中的 Greet 函数
    } else {
      log.Println("Greet function not available.")
    }
}
```

我尝试修改了函数中的打印内容：

```go
//export Greet
func Greet(name string) {
    fmt.Println("Hello,", name, "from the plugin v1!")
}
```

我测试后发现，输出显示的确监听到了 `.so` 的更新，但在重新载入后，打印的依旧是之前版本的信息。

如果你执着于 plugin 实现热更新，或许还有一个方法可尝试。既然不能卸载，那可以直接加载不同名的 `.so` 库，替换掉原来的插件。考虑它只能存在于实验中，我就不继续尝试了。

## 其他策略

不能通过 plugin 实现热更新的话，我们也有其他方式可用的，如采用服务重启或者利用微服务架构来减少更新对用户的影响。

**快速重启**

通过优化应用的启动时间和状态恢复逻辑，实现快速重启，从而减少服务不可用的时间。

**微服务架构**

将应用分解为多个小型服务，每个服务独立部署和更新。这样，更新某一部分的服务时，只会影响到该服务，而不会影响到整个应用。这也算是另一种程序上代码热更新了。

还可以与其他策略配合，如下是一些主流的思路。

**代理和版本控制**

使用代理服务器来控制流量，根据请求的版本号动态地路由到不同版本的服务实例。这样可以同时运行多个版本的服务，并逐渐将用户流量迁移到新版本，实现无缝更新。

**容器编排**

利用 Docker、Kubernetes 等容器和编排工具可以更容易地实现服务的滚动更新，尽管这不是热更新的传统意义，但它提供了类似的用户体验，减少了更新过程中的停机时间。

## 总结

综上所述，Go 在设计上不是为热更新而设计的，它的 plugin 系统确实很鸡肋。

如果要实现热更新，通过一些通用策略和工具，还是可以实现类似热更新的效果，尤其是在微服务架构中。可根据具体的应用场景和需求，选择最合适的更新策略。


