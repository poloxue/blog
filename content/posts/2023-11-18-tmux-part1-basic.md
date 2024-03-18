---
title: "我的终端环境：Tmux Part1 - 快速一览"
date: 2023-11-17T19:24:31+08:00
draft: false
comment: true
description: "本文将主要介绍 tmux 的快速一览。"
---

本文开始，我将用一个系列介绍如何高效使用 [Tmux](https://github.com/tmux/tmux)。

关于 Tmux 教程有不少，有一些写的非常不错，我也来尝试下这个主题吧。

本篇博文是系列第一篇，目标是介绍我使用 Tmux 的快速一览。我将只演示效果，不介绍细节。后续文章再逐步介绍，打造一套高效的 Tmux 工作环境。

{{< image "2023-11/2023-11-18-tmux-part1-basic-02.gif" >}}

## 何为 Tmux？

Tmux 是一款终端复用器，即 terminal multiplexer，它能实现将会话与终端的解绑，同时支持管理多个会话和窗口。


从如上的介绍中，能了解到 tmux 的两个核心能力，即会话与终端的解绑、会话窗口的管理。

## 快速安装

```bash
❯ brew install tmux
```

验证安装是否成功：

```bash
❯ tmux -V
tmux 3.3a
```

## 会话与终端解绑

简言之，即使终端窗口关闭，如果 tmux 没有停止则会话不停。

是不是想到了另外一个类似能力的 Shell 命令 - screen？但 Tmux 比它更加强大。

基于 Tmux 的这个能力，我们可以将一些后台任务放在 tmux 中进行，如此一来，即使如 ssh 断连，任务也还在继续运行。

演示案例，使用 tmux 命令开启一个 tmux 会话，执行 top 命令，detach 会话后，重新使用 tmux attach 进入会话。

效果如下所示：

{{< image "2023-11/2023-11-18-tmux-part1-basic-01.gif" >}}

我们会发现 top 命令还在运行中。

## 会话窗口的管理

Tmux 支持同时开启多个会话和窗口，实现一个终端多会话多窗口的效果。

利用 Tmux 的这个特点，我们可以创建一些会话面板，长期运行多个任务，如创建一些监控面板。

再者，如果你习惯于在 terminal 上 coding，还可通过这种方式管理常驻我们的日常开发工作区。

{{< image "2023-11/2023-11-18-tmux-part1-basic-03.gif" >}}

## Tmux 与 Vim 集成

对于平时使用 Vim 或 Neovim 为编程环境的朋友而言，Tmux 与 Vim 的集成能最大化提高日常工作的生产力。

如我们可以将两者的导航快捷键打通，实现 CTRL+hjkl 组合键在 Vim 和 Tmux 的 panes 之间的快捷跳转。

{{< image "2023-11/2023-11-18-tmux-part1-basic-04.gif" >}}

## 剪贴板共享

我们还可以将 Vim、Tmux 甚至系统的剪贴板打通，继续将我们的的 CV 大法发扬广大。

{{< image "2023-11/2023-11-18-tmux-part1-basic-05.gif">}}

补充说明，Tmux 如果没有配置好，使用起来很别扭，如会话窗口间的切换、拷贝粘贴、缩放等能力，还有默认的快捷键配置，要不是使用体验差，要不就是不支持。

估计有不少朋友在简单用了 Tmux 几天就放弃了，或只是当成 screen 使用。

## 窗口管理器 tmuxifier

对于一些固定的会话窗口布局，我们还可以通过类似于 tmuxifier 或 tmuxinator 以配置方式固定布局，这样即使 tmux 关闭重启，也可一键开启窗口布局。

一个示例，配置一个博客写作工作空间，效果如下所示：

{{< image "2023-11/2023-11-18-tmux-part1-basic-06.gif" >}}

配置一个主的 pane 用于 vim 编写博客，底部的两个 pane，一个用于执行命令，一个用于运行启动 hugo server 实时查看博文实时效果。类似于 web 开发中常用的三个 pane。

## 窗口布局持久化

Tmux 支持插件机制，同样有会话持久化的插件。它能实现在 tmux 重启后，恢复会话的最新状态。

不过，我确实不喜欢这种方式，电脑或服务器重启后，我希望由我决定重新开启什么样的环境。否则，久而久之，持久化的会话可能很多，杂乱无章。

## 快速配置方式

对于懒人而言，如果我就是不想花时间去研究这些配置，只希望有一个现成的配置，知道如何使用就行了，该如何做呢？不少人其实并不想了解复杂的配置。但问题是，不配置的，Tmux 的默认配置确实是难以上手。

网上有人会提供了现成的配置，较流行的就是这个 oh-my-tmux，访问 GitHub 地址 [oh-my-tmux](https://github.com/gpakosz/.tmux)。它默认配置出了一个易用的 Tmux 版本，前面我们提到的能力，基本在 oh-my-tmux 都有提供，如果希望自定义，也可很方便地修改。

我的博文：[我的终端环境：Tmux Part1 - 快速一览](https://www.poloxue.com/posts/2023-11-18-tmux-part1-basic/)  和 Tmux 官方文档：[Tmux official documentation](https://github.com/tmux/tmux/wiki)

