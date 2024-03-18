---
title: "如何评价 Golang 开源库 Cobra"
date: "2019-12-03T05:30:11+08:00"
comment: true
draft: false
tags: ["Golang"]
---

开源库 Cobra 是 Golang 下用于创建命令行应用的框架。 它也是 k8s、hugo 等开源项目都在用框架。[GitHub 项目地址](https://github.com/spf13/cobra)

## 命令行应用

对于 Golang 而言，一般写命令行应用，如果要求不是太多，直接用 flag 标准库就够了，毕竟像 Go 命令也是通过 flag 包实现的，完全能够驾驭。前段时间写了篇文章：[Go 命令行解析 flag 包之通过子命令实现看 go 命令源码](https://www.poloxue.com/posts/2019-11-23-commandline-tool-flag-in-golang/)

命令行应用可以和 web 应用做类比，就像 web 只有路由，就可以支持实现一个复杂的 web 应用，同样 flag 提供的基本功能，也足够写出足够复杂的命令行应用了。

但不是任何人都想去了解 flag，或者裸写一个命令行应用。

为什么呢？

因为命令行的有些内容还要处理，如，默认没有子命令的一套实现规范，不支持参数校验、不支持帮助信息模板配置。没有对一些标准提供默认支持，如 POSIX 标准，没有 -v，--version 和 -h、--help 的默认支持等。 

一套完整的命令行框架应用就要这些能力，能提供一套最佳实践。Cobra 就是这样一套框架。

## Cobra

Cobra 支持的功能非常多，一些非常出名的开源项目，比如 k8s、hugo 等都在使用它。GitHub 的说明已经介绍了它丰富的能力。

概述而言，它支持子命令，posix 规范的 Flag，嵌套的子命令，支持全局、局部和级联的选项，支持 bash 自动补全，便捷的参数校验。具体可查看 GitHub [README](https://github.com/spf13/cobra)，或

它还提供了一套脚手架，能便捷地创建一个命令行应用，就像写 web 应用一样，快速创建一个 handler、Controller。

当然，如果只是写个简单小工具，连子命令都没有，Flag 选项又少的可怜。flag 就足够了，没有必要依赖它。
 
## 使用演示

使用 Cobra 创建一个简单的命令行应用，[官方文档案例](https://cobra.dev/)。

### 安装

博文地址：[Golang 开源库 Cobra 介绍与使用](http://localhost:1313/posts/2019-12-03-what-is-cobra/)，知乎问题：[如何评价 Cobra （Golang 库）？](https://www.zhihu.com/question/358956995/answer/919748685)

