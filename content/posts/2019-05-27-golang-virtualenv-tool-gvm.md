---
title: "Go 虚拟环境管理工具 gvm"
date: 2019-05-27T12:57:35+08:00
draft: false
comment: true
tags: ["Golang"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-08-golang-virtualenv-tool-gvm-01.png)

本文谈下我对 Go 版本管理的一些想法。让后，我将介绍一个小工具，gvm。这个话题说起来也很简单，但如果想用的爽，还是要稍微梳理下。

# 背景介绍

Go 的版本管理，并非包的依赖管理，而且关于如何在不同的 Go 版本之间切换。平时的工作中，正常情况，我们不会遇到这样的需求，所以可能并不明白它的价值。

简单说下我写这篇文章的背景吧。

最近几周，Go 最重要的一则消息应该莫过 9月份 Go 1.13 的正式发布。它的相关升级可查看 [Go 1.13 正式发布，看看都有哪些值得关注的特性](https://studygolang.com/topics/10021) 或官方 [Go 1.13 Relase Notes](http://docs.studygolang.com/doc/go1.13)。

对于一名 gopher 而言，可能早已按捺不住自己那颗躁动的心，想尽快体验下新版的升级项。但问题是，切换至新版 Go 通常会遇到一些问题，比如不同版本的环境配置，安装的辅助工具和程序包在不同版本下可能会存在兼容或被覆盖等问题。

我自然就希望有一套方案可以帮助我完成 Go 版本的切换，实现不同版本间环境的完全隔离。

# 思考方案

谈到环境隔离，有很多方案可供选择，如多主机、虚拟机、容器等技术。这些听起来都挺不错，都能实现需求。但如果只是为了 Go 版本管理，完全可以自己实现。

多版本切换，主要是不同版本环境变量的隔离。Go 1.10 之前，我们关心的变量有 GOROOT、GOPATH 和 PATH。Go 1.10 之后，GOROOT 已经默认为 go 的当前安装路径，只要考虑 GOPATH 和 PATH 即可。

最近，刚答过一个关于 Go 环境变量的问题，[查看回答](https://www.zhihu.com/question/346680587/answer/828786340)。其中对每个变量的作用进行了比较细致的描述。

# 如何实现

现在，我要实现我自己电脑上的两个版本的 Go 自由切换，该如何做呢？

假设它们分别位于 ~/.goversions/sdk/ 目录下的 go1.11/ 和 go1.13/。我现在要启用 go 1.11，运行如下命令即可：

```bash
$ export PATH=~/.goversions/sdk/go1.11/bin/:$PATH
```

此时，GOROOT 已经自动识别，为 ~/.goversions/sdk/go1.11/。Go 相关的工具链，源码，标准库都在这个目录下。

但除 Go 本身相关的，还有其他第三方标准库、编译生成的库文件等内容，它们都位于 GOPATH 下，如果不设置，默认为 ~/go，在切换多版本的时候，就会产生混乱。我们可以为每个版本单独设置个 GOPATH。

如 go1.11，设置 GOPATH 为 ~/.goversions/gopath/go1.11-global/。

```bash
$ mkdir ~/.goversions/gopath/go1.11-global/
$ export GOPATH=~/.goversions/gopath/go1.11-global/
```

一个独立的环境创建好了。

如果现在要切换至 go 1.13，几个命令即可搞定。

```bash
$ export PATH=~/.goversions/sdk/go1.13/bin/:$PATH
$ mkdir -pv ~/.goversions/gopath/go1.13-global/
$ export GOPATH=~/.goversions/gopath/go1.13-global/
```

切换成功。

虽然，已经实现了需求，但总觉得用起来非常不爽。为了操作方便，其实可以把上面的思路提炼成 shell 脚本，整理成一套工具。

是不是蠢蠢欲动，想试一下？

但很遗憾，已经没这个机会了，因为这个工具已经有人开发了，思路类似，但却比这里描述的要强，它就是 gvm, 地址 [moovweb/gvm](https://github.com/moovweb/gvm)。

# 什么是  gvm

gvm，即 Go Version Manager，Go 版本管理器，它可以非常轻量的切换 Go 版本。对比其他语言，通常也有类似的工具，如 NodeJS 的 NVM，Python 的 virtualenv 等。

gvm 不仅包含上面提到的版本切换，还可以直接通过源码编辑安装任意版本的 Go，当然最好是 1.5 及之后版本，原因后面解释。

一件比较尴尬的点，gvm 产生背景并非是为了 Go 在不同版本间的切换，开发团队当初开发这个工具主要为了解决项目的依赖问题，通过切换环境实现包依赖的切换。下面，我会演示如何做到这一点。

但问题是，现在 Go 的依赖管理已经日趋完善，官方的 go module 也越来越好用，GOPATH 在被逐渐弱化，gvm 似乎也就只剩下帮我们快速体验不同 Go 版本的功能还有点价值。

废话说了那么多，开始正式体验下这个工具吧。

# 如何安装

安装很简单，只要如下一行命令即可搞定。

```bash
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

输出显示：

```bash
Cloning from https://github.com/moovweb/gvm.git to /home/vagrant/.gvm
which: no go in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin)
No existing Go versions detected
Installed gvm v1.0.22

Please restart your terminal session or to get started right away run
 `source /home/vagrant/.gvm/scripts/gvm`
```

安装完成！

重启控制台或执行 source $HOME/.gvm/scripts/gvm 即可启用 gvm。

提醒下，不同操作系统还需要相应的依赖项要装，具体查看 [项目说明](https://github.com/moovweb/gvm#mac-os-x-requirements) 的介绍。 这里面没有提到 Windows，不知道可不可用。

# gvm 安装 Go

gvm 通过从 github 下载源码编译 Go 的安装。而版本则是基于源码中的 tag。因为 1.5 版本及之后，Go 已经实现了自编译，因而要使用 gvm 安装 Go，我们要提前有可用的 Go 环境。

好在，gvm 也提供直接通过下载二进制包的方式安装 Go。

```bash
❯ gvm install go1.19 --binary
Installing go1.19 from binary source
```

Go 安装完成，就可以使用 gvm 随意安装切换任意版本的 Go 了。

```bash
$ gvm install go1.11
```

等待运行完成即可。

首次安装的时间可能会比较久，主要取决于你的网络，因为第一次需要从 github 下载源码。

# 查看版本

首先，查看下我的系统已经安装哪些 Go 版本有哪些吧，相关命令 gvm list。

```go
$ gvm list

gvm gos (installed)

   go1.11
   go1.12
   go1.13
   go1.13beta1
```

安装了 4 个版本，其中，go1.13beta1 是非稳定版本，所以说，如果我们想尽快尝试 go 的新特性，gvm 还是很便捷的。

除了查看已安装的版本，还可以通过 gvm listall 查看所有版本，版本来源于源码中的 tag 标签。

```go
$ gvm listall

gvm gos (available)

   go1
   go1.0.1
   go1.0.2
   go1.0.3
   go1.1

   ...

   go1.13
   go1.13beta1
   go1.13rc1
   go1.13rc2
```

但这个操作在 mac 上无法执行，gvm 的实现中用到了 Linux 的 sort 命令，它与 mac 上的 sort 不兼容。

怎么解决？

安装个软件 coreutils， 它之中有个 qsort 命令可用。通过 brew install coreutils 可直接安装。然后，修改下文件 $HOME/.gvm/scripts/function/tool，将其中的 sort 修改为 qsort 即可。

# 选择版本

选择启用的版本就非常简单了。如下：

```bash
$ gvm use go1.11 [--default]
```

启用成功后，可以通过 go version 和 go env 确认下。如果想默认一个版本，加上 --default 设置即可。

# 包环境管理

gvm 除了 Go 版本的管理，还可以管理包环境，相关命令是 pkgenv 和 pkgset。如果没使用包依赖管理工具，它也是挺方便的。

演示个例子，假设我们要创建一个新的项目 blog，可提前创建相应的环境。

```bash
$ gvm pkgset create blog  # 创建
$ gvm pkgset use blog     # 启用
```

闲杂，我们通过 go get 安装的包都会默认在 blog 环境下。基于的原理是 go get 默认会把安装的放在 GOPATH 中的第一个目录下。

好了，就介绍这么多吧。有兴趣的朋友可以再研究研究。毕竟在有了 go mod 之后，这个功能以后是基本不会用了。

# gvm 目录结构

gvm 是 shell 编写，默认是安装在 $HOME/.gvm/ 目录下。查看下它的目录结构会有助我们了解它的实现。

其中几个主要的目录，如下：

```bash
archive             # go 源码
bin                 # gvm 可执行文件
environments        # 不同环境的环境变量配置
scripts             # gvm 的子命令脚本
logs                # 日志信息
pkgsets             # 每个独立环境 gopath 所在路径
```

在研究了 gvm 的实现后，我们会发现，这一套思路其实也适用于其他很多工具的版本管理。如果之后再遇到同样的需求，即使我们没有现成的工具，自己实现一套也是可以的。

# 总结

本文从我的需求出发，引出了如何灵活地进行管理 Go 版本的话题。

以往的经验告诉我，既然其他语言都有工具实现这样的需求，Go 也应该有。搜索了下，找到了 gvm。虽说我在使用它的时候，发现了一些 bug 与体验不好的地方，但总体而言，已经足够满足我的需求。

# 参考

[Go 语言多版本安装及管理利器 - gvm](https://dryyun.com/2018/11/28/how-to-use-gvm/)
[moovweb/gvm](https://github.com/moovweb/gvm)
[gvm + go mod](https://medium.com/golang-%E7%AD%86%E8%A8%98/gvm-go-mod-492a54c15c41)

博文地址：[Go 虚拟环境管理工具 gvm](https://www.poloxue.com/posts/2019-05-27-golang-virtualenv-tool-gvm)
