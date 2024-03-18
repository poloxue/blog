---
title: "你真的不用再设置 GOROOT 了"
date: 2019-11-06T12:42:29+08:00
draft: false
comment: true
tags: ["Golang"]
---

为什么不再需要设置 `GOROOT` 呢？推荐读两篇英文文章，我意译了下，将它们放在了一篇里。

[第一篇](https://dave.cheney.net/2013/06/14/you-dont-need-to-set-goroot-really)是关于 Go 1.10 之前，怎么设置 `GOROOT`，发表与 2013 年。[第二篇](https://go-review.googlesource.com/c/go/+/42533/) 是从 Go 1.10 开始，如何处理 `GOROOT`，时间是 2018 年，Go 源码提交日志。这篇非常短小。

读完后，你会发现，大多数情况下，你都不用手动设置 `GOROOT` 了。

# 第一篇

作者：Dave Cheney  | 地址：[you-dont-need-to-set-goroot-really](https://dave.cheney.net/2013/06/14/you-dont-need-to-set-goroot-really)

一篇小短文，解释了为什么在编译和使用 Go 时，不需要设置 `GOROOT`。

## 概要性介绍

一般来说，在 Go 1.0 之后，编译和使用 GO 不再需要设置 `GOROOT`。事实上，如果你的电脑上存在多个版本的 Go 语言环境，设置 `GOROOT` 可能产生一些问题。

`GOPATH` 仍然需要设置。

从 Go 1.0 开始，`GOPATH` 就被强烈推荐。随着 Go 1.1 的发布，`GOPATH` 已经是强制性的了。

## 为什么不再要设置 `GOROOT`？

**谈些 Go 环境变量的历史吧！**

Go 的资深老前辈们可能还记得，曾经的 Go 不仅要设置 `GOROOT`，还需要设置 `GOOS` 和 `GOARCH`。之所以要设置 `GOROOT`，是因为 Make 在编译构建的时候，引入了 `GOROOT` 中的内容，要提前设置 `GOROOT` 作为了它们的基本路径。

随着 `go tool` 的引入，Go 1.0 之前，`GOOS` 和 `GOARCH` 已经变成可选了，因为构建脚本已经能自动检测出系统类别和 CPU 架构。在 Go 1.0 的发布后，引入了 cmd/dist 引导构建工具，`GOOS` 和 `GOARCH` 真正意义上是可选项了，仅仅在交叉编译时才会用到。

**不需要设置 `GOOS` 和 `GOARCH`，那 `GOROOT` 呢？**

`GOROOT` 定义为指定安装 GO 的根目录。在之前的 Makefile 中，引入其他 Makefile 时，将它作为基础路径。而且，Go 1.0 之后，`go tool` 利用它查找 Go 编译器（保存在 `$GOROOT/pkg/tool/$GOOS_$GOARCH`）和标准库（在 `$GOROOT/pkg/$GOOS_$GOARCH`）。如果你是一名 Java 开发者，可以将 `GOROOT` 理解为 `JAVA_HOME`。

源码编译 Go，`GOROOT` 将自动发现（all.bash 的上级目录），然后设置到 go 工具链。

如下命令查看：

```bash
$ echo $GOROOT

$ go env
/home/dfc/go
```

从 golang.org 下载的二进制包或者系统方式安装的 Go 环境，也已在工具链中设置了正确的 GOROOT。

一个例子，比如 Ubutun 12.04 下，安装了 Go 1.0。

```bash
$ dpkg -l golang-{go,src} | grep ^ii
$ go
/usr/bin/go
$ go env GOROOT
/usr/lib/go
```

我们可以看出，Go 工具链被安装在了 `/usr/bin/go` 下，`GOROOT` 内置为 `/usr/lib/go`

**为什么不应该设置 `GOROOT`**

我们不应该设置 `GOROOT`，是因为 Go 工具链已经内置了正确的值。

设置 `GOROOT` 将会覆盖掉保存在 go 工具链中的默认值，可能会导致 go 执行不同版本的编译器和标准库文件。

两种情况下，你需要设置 `GOROOT`。在官方的 [安装介绍](http://golang.org/doc/install#install) 有相关的描述。

- 如果你是 Linux、FreeBSD 或者 OS X 用户，下载了 zip 和 tarball 的二进制包安装环境。这些二进制的默认环境位于 /usr/local/go，建议你将 Go 安装到这个位置。如果选择不这么做，就必须设置到你指定的目录下。
- 如果你是 Windows 用户，使用 zip 二进制包安装，默认的 `GOROOT` 在 C:\Go 目录下。如果你将 Go 安装在其他位置，请设置 `GOROOT` 到指定的目录。

## 其他细节

本文已经介绍了当通过源码编译 Go 环境的时候，`GOROOT` 如何自动发现的。但如果 `GOROOT` 与 all.bash 所在位置并不匹配呢？比如，在临时目录下编译 Go 环境，如何正确地设置 `GOROOT` 呢？答案是使用 `GOROOT_FINAL`，它将被用于覆盖自动发现的 `GOROOT`，设置到 GO 工具链中。

举个例子，在 Debian/Ubuntu 上，构建程序会将 `GOROOT_FINAL` 的值设置为 /usr/lib/go。保持 `GOROOT` 是未设置状态，使构建编译愉快地执行。构建完成后，将 Go 工具链安装到 /usr/bin 目录下，编译器、源码和包安装到 /usr/lib/go 下。

## 注意点

如果使用二进制包安装 Go 环境，有些特殊情况需要处理，本文已经作了相关描述。

虽然构建系统能自动检测，但如果 all.bash 的父级目录不满足 `GOROOT` 要求，也需要另外处理。

# 第二篇

翻译自 Go 的提交日志，地址：[use os.Executable to find GOROOT](https://go-review.googlesource.com/c/go/+/42533/)。

Go 1.10 开始，通过 `use os.Executable` 查找 `GOROOT`。

之前，我们是通过 make.sh 编译构建 `GOROOT`，但如果将整个目录移动到新的路径下，这会使 Go 工具链无法正常工作。

如何解决这个问题呢？

一是可以将源码重新编译，但如果新位置在其他用户的目录下，可能就无法这么操作了。

二是，通过设置 `GOROOT` 环境变量的方式解决，但另外设置 `GOROOT` 是不推荐的，因为它可能使一个环境下 `go tool` 使用了另一个环境下 `compile`。

这次的修改，`go tool` 将通过相对路径的方式确定 `GOROOT`，通过使用 `os.Execute` 函数。同时，还会检查 `$GOROOT/pkg/tool` 目录是否存在，以避免下面的两种情况。

```bash
$ ln -s $GOROOT/bin/go /usr/local/bin/go
```

和

```bash
$ PATH=$HOME/bin:$PATH
$ GOPATH=$HOME
$ ln -s $GOROOT/bin/go $HOME/bin/go
```

另外，如果当前的执行路径并不在 `GOROOT` 下，将会通过软连接找到真正的命令的位置，检查这个路径是否是 `GOROOT`。
