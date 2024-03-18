---
title: "Go 命令行解析 flag 包之快速上手"
date: 2019-11-23T16:21:33+08:00
draft: false
comment: true
tags: ["Golang"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-23-commandline-tool-flag-in-golang-01.png)

本篇文章是 Go 标准库 flag 包的快速上手篇。

## 概述

开发一个命令行工具，视复杂程度，一般要选择一个合适的命令行解析库，简单的需求用 Go 标准库 flag   就够了，flag 的使用非常简单。

当然，除了标准库 flag 外，也有不少的第三方库。比如，为了替代 flag 而生的 [pflag](github.com/spf13/pflag)，它支持 POSIX 风格的命令行解析。关于 POSIX 风格，本文末尾有个简单的介绍。

更多与命令行处理相关的库，可以打开 [awesome-go#command-line](https://awesome-go.com/#command-line) 命令行一节查看，star 最多的是 [spf13/cobra](https://github.com/spf13/cobra) 和 [urfave/cli](https://github.com/urfave/cli) ，与 flag / pflag 相比，它们更加复杂，是一个完全的全功能的框架。

有兴趣都可以了解下。

## 目标案例

回归主题，继续介绍 flag 吧。通过案例介绍包的使用会比较直观。

举一个例子说明吧。假设，现在要开发一个 Go 语言环境的版本管理工具，gvg（go version management using go）。

命令行的帮助信息如下：

```
NAME:
   gvg - go version management by go

USAGE:
   gvg [global options] command [command options] [arguments...]

VERSION:
   0.0.1

COMMANDS:
   list       list go versions
   install    install a go version
   info       show go version info
   use        select a version
   uninstall  uninstall a go version
   get        get the latest code
   uninstall  uninstall a go version
   help, h    Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```

这个命令不仅包含了全局的选项，还有 8 个子命令，部分子命令支持参数和选项。暂时，子命令的选项参数先不列出来了，实现时再看。

接下来，我们试着通过 flag 实现这个效果。本文只介绍 GLOBAL OPTIONS（全局选项）的实现。

> 如果想了解什么是 Go 语言环境的版本管理，可以查看 [如何灵活地进行 Go 版本管理](https://mp.weixin.qq.com/s/OMZ3epByc_bQoMIr4OoetQ) 一文。


## 选项表示

最简单的命令不需要任何参数和选项，复杂一点，要支持参数和选项的配置。gvg 没有全局参数，或者说全局参数是子命令，全局选项有 `--help -h` 和 `--version -h`。

一个选项在 flag 包中用一个 `Flag` 表示，那 `-h` 可以用一个 `Flag` 表示。一个选项通常由几个部分组成，如名称、使用说明和默认值。如果将 `-h` 用代码表示，如下：

```go
h := flag.Bool("h", false, "show help")
```

定义了一个布尔类型的 `Flag`，名为 `h`，默认值是 false，使用说明为 "show help"。变量 `h` 是一个布尔型的指针，通过它可以取出命令行传入的值。

除了使用 `flag.Bool`，还可以使用另外一种方式，`Flag.BoolVar` 定义一个 `Flag`。我们可以用这种方式定义 `-v` 选项。

代码如下：

```go
var v bool
flag.BoolVar(&v, "v", false, "print the version")
```

最后的三个参数含义与 `flag.Bool` 相同，主要区别在值的获取方式，`flag.BoolVar` 是通过将变量地址传入获取值。从经验来看，第二种方式使用的较多，或许因为第一种方式会发生变量逃逸。

## 更多类型

除了布尔类型，`Flag` 的类型还有整数（int、int64、uint、uint64）、浮点数（float64）、字符串（string）和时长（time.Duration）。

假设 gvg 的案例中，支持配置文件选项 `--config-path`。实现代码如下：

```go
var configPath

flag.StringVar(&configPath, "config-path", "", "config file path")
```

通过 `StringVar` 定义了新的 `Flag`。使用方式与 `BoolVar` 相同，最后的三个参数分别是选项名称、默认值和使用说明。

虽然 flag 支持的内置类型并不多，但已经满足大部分需求了。如果有自定义的需求，也可以扩展新的类型实现，这部分内容下篇介绍。

## 长短选项

现在已经完成了 `-h` 和 `-v` 两个选项，但目标是 `-v --version` 和 `-h --help`，即同时支持长短选项。

一个 `Flag` 应该有长短两种形式，但 flag 包并不支持这种风格，需要曲线救国才能实现。（注：本文开开头提到的 pflag 支持。）

这里以 `-v --version` 为例，代码如下：

```go
flag.BoolVar(&v, "v", false, "print the version")
flag.BoolVar(&v, "version", false, "print the version")
```

定义了两个 `Flag`，同时绑定到了一个变量上。这种效果只能用 `flag.BoolVar` 方式定义新的 `Flag`，`flag.Bool` 没办法做到将同一个变量同时绑定两个 `Flag`。

但其实这种也有缺点，先不说了，后面介绍帮助信息打印时就明白了。

## 命令行解析

定义好所有 `Flag`，还需要一步解析才能拿到正确的结果。这一步非常简单，调用 `flag.Parse()` 即可。

如下是完整的代码：

```go
package main

var h *bool
var v bool

func init() {
	flag.BoolVar(&h, "h", false, "show help")
	flag.BoolVar(&h, "help", false, "show help")
	flag.BoolVar(&v, "v", false, "print the version")
	flag.BoolVar(&v, "version", false, "print the version")
}

func main() {
	flag.Parse()
	fmt.Println("version", v)
	fmt.Println("help", h)
}
```

通过 `flag.Parse()` 解析完成，打印下 `v` 和 `h` 变量，确认下是否成功获取到了值。

到此，代码就告一段落了，现在将它编译为 gvg 命令吧。

## 使用命令

在正式使用命令前，先介绍下 flag 的语法。官方文档说明，命令行中 flag 选项的使用语法有如下几种形式。

```bash
-flag
-flag=x
-flag x // 非布尔类型才支持这种方式
```

但其实，-- 也是支持的。因此，上面才可以实现 `--version` 的曲线救国。

使用下这个命令，将 `help` 设置为 `false` 和 `version` 设置为 `true`。我尽量把所有可能的写法都列出来。

```bash
$ gvg -v
$ gvg -version -h=false  # 单个 - ，即 -version 支持
$ gvg --version=true --help=false
$ gvg --version=1 --help=0
$ gvg --version=t --help=f
$ gvg --version=T --help=F
$ gvg --version true --help true # 写法错误，因为无法识别出是 bool 值，还是参数或子命令
$ gvg -vh  # 不支持这种风格
```

执行命令，输出结果：

```
version true
help false
```

到这里，flag 的快速入门就介绍完了。参数留在子命令的时候介绍。

## 命令行风格

由于一些历史原因，Unix 出现过很多不同的分支，命令行的风格也因此有很多标准，比如：

- Unix 风格，选项采用单 `-` 加一个字母，比如 `-v`，短选项就是它，优点是足够简洁;
- BSD 风格，选项没有 `-`，没有任何的前缀，不知道有参数的情况怎么处理，没有研究;
- GNU 风格，采用 `--`，如 `--version`，长选项，扩展性好，但是要多打几个字母;

在网上找到一个搞笑漫画。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-23-commandline-tool-flag-in-golang-02.jpeg)

查看系统进程有两种写法， `ps aux`（BSD 风格） 和 `ps -elf`（Unix 风格）。之前，我一直很郁闷为什么有这个区别。现在算是明白了。哈哈。

POSIX 的命令行风格算是取长补短的集合吧。什么是 POSIX 风格？可以查看这篇文档 [命令参数语法](http://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html)。它同时提供了长短选项的标准。

要明白的是，标准终究只是标准，很多命令其实并不遵循它。但自己在设计命令行规范的时候，最好还是要有一套标准，而参考最统一的标准肯定是没错的。

## 总结

本文介绍了 Go 中 flag 包的使用，一般的场景已经足够使用了。

博文地址：[Go 命令行解析 flag 包之快速上手](https://www.poloxue.com/posts/2019-11-23-commandline-tool-flag-in-golang/)
