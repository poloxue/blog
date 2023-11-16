---
title: "我的终端环境：无用小知识 - neofetch/fastfetch"
date: 2023-11-14T17:56:34+08:00
draft: true
comment: true
description: "本文接着上文，介绍终端启动消息的配置，让你的终端比之前文更加绚丽多彩。"
---

本文接着上文，将介绍如何使用 fetch 配置终端启动消息。

## 前言

你是否在终端上看到过类似如下的信息？

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-01.png" >}}

我在刚开始讲终端环境这个系列，就有小伙伴在我的视频下 show 了他的终端。

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-02.webp" >}}

要实现这个终端效果，要依赖一种名为 fetch 的程序。

### 什么是 fetch？

所谓 fetch，是指一类系统信息收集的脚本，显示系统摘要信息（软硬件信息），例如发行版、内核、版本、桌面环境、窗口管理器等。它们通常系统的终端上使用，提示我们在哪里工作。

其中，较为出名的 fetch，有诸如 [neofetch](https://github.com/dylanaraps/neofetch)、[pfetch](https://github.com/dylanaraps/pfetch)、[fastfetch]() 等。

### 一键安装

```zsh
brew instal neofetch pfetch fastfetch
```

### 更多 fetch

如果希望发现更多关于 fetch 的资源，查看 [awesome-fetch](https://github.com/beucismis/awesome-fetch)。没错，fetch 这个系列也有一个 awesome 系列。

## 开始正文

让我们开始正式介绍 fetch 的使用方法。

今天呢，将主要的三个 fetch 分别是，neofetch、pfetch 和 fastfetch，有的复杂强大，有的非常简单，有的高性能。

它们是各有各的特点。

## neofetch

neofetch 是一款用 bash script 实现，快速且高度可定制的 fetch。它支持展示 ASCII 和 图片，在 Linux、BSD、Mac OS X 和 Window 等系统上可运行。它的 github star 数量也是所有 fetch 中最多的。

要说它的缺点，一是性能低，因为是 bash script 实现，性能不高；二是已停止维护，好像 bug 已停止修复。

### 使用

我将以 Mac OS X 演示其使用。

简单实用，直接输入以下命令：

```
> neofetch
```
 
效果如下：

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-03.png" >}}

- 配置

```
brew install fastfetch
```

- iterm2 背景图设置

https $(https ://api.github.com/repos/poloxue/images/branches/main | jq -r '.commit.commit.tree.url') | jq '.tree[].path'
