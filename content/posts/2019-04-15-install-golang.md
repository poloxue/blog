---
title: "详细聊聊如何安装 Go"
date: 2019-04-15T20:05:26+08:00
draft: false
comment: true
tags: ["golang"]
---

本篇文章进入 Go 的开发环境搭建系列。

我们知道，编写任何语言的代码都离不开两样工具，语言开发包和代码编辑工具。

今天先来聊聊如何安装 Go。

我们或许都会觉得这种事非常简单，不值得写篇文章介绍。最初我也是这么想的。但深入了解下来，渐渐感觉这也是一件很有意思的事情。

# 如何安装

和其他语言的安装类似，Go 的安装我们也可以采用三种方式进行，从简单到复杂依次是通过系统方式安装、官方二进制包安装和源码编译安装。

## 系统方式

不同操作系统通常都会为 Go 提供相应的软件安装方式。这种方式很大程度上简化了安装过程，能为我们省去一些繁杂的步骤。下面分别介绍下不同系统下的安装方式：

**windows**

在windows下，软件安装通常可通过下载类似 setup.exe/msi 软件包来操作。按照导航的提示，不断执行 "下一步" "下一步" 即可完成。访问 [下地地址](https://zhuanlan.zhihu.com/p/62148085) 将看到如下内容：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/install-golang-01.png)

选择其中的 "Microsoft Windows" 下载 windows 安装包。现在的系统基本都是64位的了，一般情况下不用考虑 32/64 位系统的问题。

下载好了安装包，点击启动执行，接下来的步骤就是按导航提示一步步操作即可。有一点要注意的是，GO的默认安装在 C:\GO，如果要修改默认安装路径，在见到如下界面时重新选择。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/install-golang-02.png)

**ubuntu/debian**

在debian或ubuntu上，我们可使用 apt-get 命令安装go。比如，在Ubuntu 16.04.5 LTS系统，使用如下命令安装：

```
sudo apt-get update // 视情况决定是否更新
sudo apt-get install golang-go
```

如果是新建的系统，建议先update下软件源。否则可能会因为某些源异常而无法顺利安装。

**centos/redhat**

在centos或redhat上，我们可以使用yum命令安装go。比如，在CentOS 7.5上，使用如下命令安装：

```
$ yum epel-release
$ yum install golang
```

先下载了epel-releaes源，可防止出现yum安装golang不支持或版本太旧的问题。

**macos**

在macos上，我们可使用pkg文件或homebrew安装go。

pkg的安装方式与windows的setup.exe/msi的类似，下载软件然后按导航 "下一步" "下一步" 即可完成。

来说说如何使用homebrew安装。和yum和apt-get不同，homebrew并非mac系统自带，我们需要先安装。进入homebrew官网，页面顶部便说明了安装的方式，命令如下：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

接着安装go，命令如下：

```
$ brew install go
```

非常简单就完成了安装。

系统安装方式的优点是简单。缺点是我们不能保证系统提供给的版本一定能满足我们的要求，比如上面ubuntu安装的go版本就较低，为go1.6。

## 二进制包

接下来说说如何使用二进制包安装。所谓二进制包，也就是已经编译好的包。这种安装方式在不同的操作系统上步骤类似，考虑到windows用户最多，就以windows为例吧。

再次进入到下载页面，在列表可如下内容。因为我用的32位windows虚拟机，下载i386的包。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/install-golang-03.png)

接着把下载的压缩包解压到某个文件夹，比如 c:\Program Files 下，进入查看，会发现其中已经包含了新的名为go的文件夹。

至此，二进制包的方式安装就完成了。因为二进制包是已编译好的软件包，所以不同系统、CPU架构下需要下载与之相应的包。

我们或许会想，就是移动个文件夹？是的，系统安装其实也就是做这些事情，不同在于系统安装在简化了操作的同时也会针对性做一些设置，比如配置好环境变量、文件分类存放等。

## 源码编译

这种安装方式的好处是与系统无关，一切控制权都掌握在自己身上，能限制我们的只有自己的能力。

上篇文章说过，go在1.5版本已经移除了源码中全部的C代码，实现了自编译。因此，我们可以用系统已有go来编译源码，从而实现新版的安装。

前面在ubuntu下，我用apt-get安装的golang比较老的go1.6版。下面通过它来编译新版go。

下载源码，最新版源码可点击 [go1.12.2.src.tar.gz](https://dl.google.com/go/go1.12.2.src.tar.gz) 下载。这里多说几句，go的源码托管在github上，地址：https://github.com/golang/go ，如想提前尝试新功能，可直接从github拉取最新的代码编译。这也是源码编译安装的一个好处。

源码下载完成后进入源码目录即可编译。注意，如果用虚拟机编译，要保证有充足的内存。
```
$ tar zxvf go1.12.2.src.tar.gz       // 解压源码包
$ cd go/src
$ ./all.sh
```

执行./all.sh即可完成编译安装，也挺简单的。这个过程会耗费一旦时间，要等待会。其实这里简化了很多细节，如果想仔细研究的话，可以去阅读官方文档 install go from source。

# 环境变量

在安装完golang后，还需了解三个环境变量，分别是GOROOT、GOPATH、PATH。下面来分别介绍一下它们的作用。

## GOROOT

GO安装的根目录。该变量在不同的版本需要选择不同的处理方式。

在 GO 1.10 之前，我们需要视不同安装方式决定是否手动配置。比如源码编译安装，安装时会有默认设置。而采用二进制包安装，在windows系统中，推荐安装位置为C:\GO，在Linux、freeBSD、OS X系统，推荐安装在/usr/local/go下。如果要自定义安装位置，必须手动设置GOROOT。如果采用系统方式安装，这一切已经帮我们处理好了。

关于这个话题，推荐阅读：you-dont-need-to-set-goroot和分析源码安装go的过程。

在 GO 1.10 及以后，这个变量已经不用我们设置了，它会根据go工具集的位置，即相对go tool的位置来动态决定GOROOT的值。说简单点，其实就是go命令决定GOROOT的位置。

关于这个话题，推荐阅读：[use os.Executable to find GOROOT](https://go-review.googlesource.com/c/go/+/42533/) 和 [github go issues 25002](http://blog.studygolang.com/2013/01/%E5%88%86%E6%9E%90%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85go%E7%9A%84%E8%BF%87%E7%A8%8B/)。

## PATH

各个操作系统都存在的环境变量，用于指定系统可执行命令的默认查找路径，在不写完整路径情况下执行命令。

以Windows为例，我之前把go安装在 C:\Program Files\go目录下，即GOROOT为C:\Program Files\go，那么PATH变量可追加上C:\Program Files\go\bin。

## GOPATH

如果有朋友了解python，可以将其类比为python的环境变量PYTHONPATH，用来设置我们的工作目录，即编写代码的地方。包也都是从GOPATH设置的路径中寻找。

在go 1.8之前，该变量必须手动设置。go 1.8及之后，如果没有设置，默认在$HOME/go目录下，即你的用户目录中的go目录下。

## 如何设置

介绍完三个变量，以我的mac为例介绍下设置方式吧。

类unix系统环境变量的设置方式都类似。使用export命令设置环境变量，并将命令加入到/etc/profile，该文件会在开启shell控制台的情况下执行。具体操作命令如下：
```
$ sudo vim /etc/profile
...
export GOROOT=/usr/local/go         // 默认位置可不用设置，1.10版本后也可以不设置
export PATH=$PATH:$GOROOT/bin
export GOPATH=/Users/polo/work/go   // 可设置多个目录
```

经过以上步骤，环境变量配置完成，如果要立刻启用环境变量，我们需要重启下控制台。接着我们可以用go env看一下变量的配置情况。

```
$ go env
GOARCH="amd64"
GOBIN="/usr/local/go/bin"
...
GOPATH="/Users/polo/Public/Work/go"
...
GOROOT="/usr/local/go"
```

# 目录结构

再简单介绍下go的目录结构。以windows为例，进入C:\Program Files\go将看到如下内容。


![](https://cdn.jsdelivr.net/gh/poloxue/images@main/install-golang-04.png)

介绍几个比较主要的目录：

- api，里面包含所有API列表，好像IDE使用了里面的信息；
- bin，里面是一些go的工具命令，主要是go、gofmt、godoc，命令使用方法后面介绍
- doc，go的使用文档，可以让我们在没有网络的情况下也可以阅读；
- src，主要是一些源码，如golang的编译器、各种工具集以及标准库的源码，

# 入门案例

介绍到这里已经差不多了，接着来写一个简单的例子，即经典的Hello World。

首先，创建一个名为hello.go的文件，后缀必须为.go，内容如下：

```go
package main

import "fmt"

func main(){
    fmt.Println("Hello World")
}
```

上面的代码主要由几部分组成，分别是

- package main，包声明，go中的文件必须属于某个包，main较为特殊，是程序入口所在；
- import "fmt"，导入fmt包，这是一种引入包的方式，接下来就可以使用fmt提供的函数变量；
- func main() {}，func关键字函数定义，main是函数名，在main包中为程序的入口；
- fmt.Println，main函数中的代码块，表示调用fmt提供的Println函数打印 字符串"Hello World"

接下来，我们可以使用 go run 执行下这段代码，如下：

```go
$ go run hello.go
Hello World
```

执行输出 "Hello World"。

# 总结

本篇文章从不同系统和不同方式角度出发，介绍了golang在各种场景下的安装方式。之后又详细介绍了几个在go中常用的环境变量，并以一个简单的例子结尾，最终完成了go的安装。

我的博文：[如何安装 Go](https://www.poloxue.com/posts/2019-04-15-install-golang/)。
