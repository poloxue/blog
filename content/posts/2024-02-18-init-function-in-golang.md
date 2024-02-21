---
title: "Go 中的 init 如何用？它的常见应用场景有哪些呢？"
date: 2024-02-18T08:00:00+08:00
draft: false
comment: true
description: "Go 中有一个特别函数 `init()` 函数，它在 Go 中扮演着一个特殊的角色，可用于包的一些初始化操作。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-08-init-function-in-golang-01.png)

> 嗨，大家好！我是波罗学。本文是系列文章 Go 技巧第十六篇，系列文章查看：[Go 语言技巧](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MzE2NTY2MA==&action=getalbum&album_id=3291066778475053060#wechat_redirect)。

Go 中有一个特别的 `init()` 函数，它主要用于包的初始化。`init()` 函数在包被引入后会被自动执行。如果在 `main` 包中，它也会在 `main()` 函数之前执行。

本文将以此为主题，介绍 Go 中 `init()` 函数的使用和常见使用场景。还有，我在工作中更多看到的是 `init()` 函数的滥用。

## `init()` 函数的执行时机

首先，`init()` 的执行时机处于包级别变量声明和 main() 函数执行之间。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-08-init-function-in-golang-02.png)

这意味着在包中声明的全局变量，如果附带初始化表达式，这些表达式将在任何 `init()` 函数执行之前进行初始化。

我们通过一个示例演示，代码如下：

```go
var Age = GetAge()

func GetAge() int {
    return 18
}

func init() {
    fmt.Printf("You're %d years old.\n", Age)
    Age = 3
}

func main() {
    fmt.Printf("You're %d years old.\n", Age)
}
```

输出：

```bash
You're 18 years old
You're 3 years old
```

从输出可知，`GetAge()` 函数作为 `Age` 的初始化函数，于 `init()` 函数前执行，赋值 `Age` 为 `3`。而 `init()` 函数于其后执行，赋值 `Age` 为 `3`。`main()` 函数则在最后执行，输出最终的 `Age` 值。

这个顺序是符合我们预期的。

## 与被引入包的 `init()` 函数

如果一个包导入了其他包，被导入包的初始化 `init()` 则会先于导入它的包的变量初始化和 `init` 函数前执行。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-08-init-function-in-golang-03.png)

举例来说明吧！

假设，我们有一个 `main` 包，它导入了 `sub` 包，并且同样有一个 `init()` 函数：

```go
// main.go

package main

import (
    "fmt"
    _ "demo/sub"
)

var age = GetAge()

func GetAge() int {
  fmt.Println("main initialize variables.")
  return 18
}

func init() {
    fmt.Println("main package init")
}

func main() {
    fmt.Println("main function")
}
```

而 `sub` 包中包含定义的 `init()` 函数
```go
// sub/sub.go

package sub

import "fmt"

var age = GetAge()

func GetAge() int {
  fmt.Println("sub initialize variables.")
  return 18
}

func init() {
    fmt.Println("sub package init")
}

// 其他可能的函数和声明
```


当你运行 `main.go` 时，输出将会按照以下顺序出现：

```
sub initialize variables.
sub package init
main initialize variables.
main package init
main function
```

这个示例清晰地展示了包的初始化顺序：首先是被导入包（`sub`）的 `init()` 函数，然后是导入它的包（`main`）的 `init()` 函数，最后是 `main` 函数。

这也确保了依赖包在使用前已经被正确初始化。

> 特别说明：
>
> `init()` 区别于其他函数，不需要我们显式调用，它会自动被 Go runtime 调用。而且，每个包中的 `init()` 只会被执行一次。
>

一个包其实可有多个 `init()`，无论是在分部在包中的同一个文件中还是多个文件中。如果分布在多个文件中，执行顺序通常是按照文件名的字典顺序。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-08-init-function-in-golang-04.png)

为说明这个问题，我们首先修改 `sub.go` 文件，内容如下：

```go
// sub/sub.go

package sub

import "fmt"

var age = GetAge()

func GetAge() int {
  fmt.Println("sub initialize variables.")
  return 18
}

func init() {
    fmt.Println("sub init 1")
}

func init() {
    fmt.Println("sub init 2")
}
```

新增一个 `sub1.go` 文件，如下所示：

```go
// sub/sub1.go

package sub

import "fmt"

var age = GetAge1()

func GetAge1() int {
  fmt.Println("sub1 initialize variables.")
  return 18
}

func init() {
    fmt.Println("sub1 init")
}
```

输出：

```bash
sub initialize variables.
sub init 1
sub init 2
sub1 initialize variables.
sub1 init
main initialize variables.
main package init
main function
```

结果符合预期。

## `init()` 的使用场景

`init()` 函数通常用于进行一些必要的设置或初始化操作，例如初始化包级别的变量与命令行参数、配置加载、环境检查、甚至注册插件等。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-08-init-function-in-golang-05.png)

项目开发中，组件依赖管理通常比较令人头疼。但一些简单的依赖关系，即使没有如 `wire` 这样依赖注入工具的加持，通过 `init` 也可管理。

### 命令行参数

对于开发一个简单的命令行应用，`init()` 和标准库 `flag` 包结合，可快速完成命令命令行参数的初始化。

```go
package main

import (
    "flag"
    "fmt"
)

var name string
var help bool

func init() {
    flag.StringVar(&name, "name", "World", "a name to say hello to")
    flag.StringVar(&help, "name", "World", "display help information")
    flag.Parse()
}

func main() {
    if help {
        fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
        flag.PrintDefaults()
        os.Exit(1)
    } 
    fmt.Printf("Hello, %s!\n", name)
}
```

以上示例中，`init()` 函数解析了命令行参数并初始化变量 `name` 和 `help` 变量。

### 配置加载

`init` 函数的领哇一个常见场景是配置加载。配置通常是程序启动时要尽早执行的操作。

例如，你有一个 web 服务，要在启动服务器前加载数据库配置、API 密钥或其他服务配置。

```go
var config AppConfig

func init() {
    configFile, err := os.Open("config.json")
    if err != nil {
        log.Fatal(err)
    }
    defer configFile.Close()
    jsonParser := json.NewDecoder(configFile)
    jsonParser.Decode(&config)
}
```

如果配置加载都出现问题，很大程度说明服务配置不正常，要立刻退出服务。我们可使用 `log.Fatal(err)` （更优雅）或 `panic(err)` 退出服务。

### 环境检查

`init()` 还可以用于检查和验证程序运行所需的环境。如，我们要确保必要的环境变量已设置，或者必要的外部服务可用。

如我们的必须依赖一个需要认证的外部服务，示例代码：

```go
func init() {
    if os.Getenv("XXX_API_KEY") == "" {
        log.Fatal("XXX_API_KEY environment variable not set")
    }

    apiKey := os.Getenv("XXX_API_KEY")
    // instantiating Component
    // ...
}
```

通过，如果要实例化的组件不需要赖加载，创建和配置验证同时 `init()` 中完成即可。

### 注册插件或服务

如果你的程序用的是插件架构，我们可以在程序启动时注册这些插件。`init()` 正可以用来自动注册这些插件。

示例代码：

```go
func init() {
    plugin.Register("myPlugin", NewMyPlugin)
}
```

Go 的数据库驱动管理可作为这种场景的典型案例。

Go 的 database 操作通常依赖 `database/sql` 包，它提供了一种通用接口与 SQL 或类 SQL 数据库交互。而具体的驱动实现（如 MySQL、PostgreSQL、SQLite 等）通常是通过实现 `database/sql` 包定义接口来提供支持。

这种架构下，`init()` 被用于驱动的自动注册。

例如，如下这个 MySQL 驱动的实现：

```go
package mysql

import (
    "database/sql"
)

func init() {
    sql.Register("mysql", &MySQLDriver{})
}

type MySQLDriver struct {
    // 驱动的实现
}
```


我们只要导入这个 database driver 包，它的 `init()` 就会被调用，将驱动注册到 `database/sql` 包中。

我们使用的时候，通过 `database/sql` 接口即可使用该 `MySQL` 驱动，而不需关心它的实现细节。

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql" // 导入 MySQL 驱动
)

func main() {
    db, err := sql.Open("mysql", "user:password@/dbname")
    // ...
}
```

通过这种方式，Go 的数据库驱动代码更加模块化和灵活性。使用方只需关心与 `database/sql` 交互即可，而不必关心驱动的实现细节。

实际的场景案例，我觉得肯定不止这么多。对于任何需要提前初始化和验证的场景，可适当考虑是否可通过使用 `init()` 来简化代码。

## 注意点

讲了那么多 `init()` 的使用，但我在平时发现，更多的时候 `init()` 函数是在被滥用。

我这里不得不提一些注意点。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-08-init-function-in-golang-06.png)

### 启动耗时

首先，由于 `init()` 函数在程序启动时自动执行，这就导致它会增加程序启动时间，特别是一些组件初始化耗时较长。

非必要场景，懒加载依然是不错的选择。

什么是必要场景呢？简单来说，如果这个操作失败了，这个程序就没有继续启动的必要了。

### 依赖关系 

还有，过多或过于复杂的 `init()` 函数可能会导致程序难以理解维护，依赖关系混乱。

这点在单体项目中体现的特别明显，所有人维护一个项目，所以依赖都加载到 `init()` 中。

如何解决呢？

如前面所有，一方面要仅在必要场景时使用 `init()` 函数初始化一些操作。

另外，有条件的话，建议尽量保持服务简单，如果依赖过多，如出现要一个服务连接多个相同组件（数据库、Redis），就是时候考虑优化系统设计了，可考虑将部分业务抽离为独立服务。

## 总结

本文介绍了到 `init()` 函数在 Go 中的特殊之处和使用方式。它提供了一种不同于其他语言的机制来初始化包，但也需谨慎使用以避免不必要的复杂性。

最后，希望这篇文章能帮助你更好地理解和使用 Go 的 `init()` 函数。

感谢阅读。

博客地址：[Go 中的 init 如何用？它的常见应用场景有哪些呢？](https://www.poloxue.com/posts/2024-02-08-init-function-in-golang)
