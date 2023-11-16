---
title: "我的终端环境：无用小知识 - pfetch/neofetch/fastfetch"
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

所谓 fetch，是指一类系统信息收集的脚本，显示系统摘要信息（软硬件信息），例如发行版、内核、版本、桌面环境、窗口管理器等。fetch 通常是在系统的终端上使用，显示我们的工作环境。

其中，较为出名的 fetch，有诸如 [neofetch](https://github.com/dylanaraps/neofetch)、[pfetch](https://github.com/dylanaraps/pfetch)、[fastfetch]() 等。

### 一键安装

```zsh
brew instal pfetch neofetch fastfetch imagemagick w3m
```

其中，`imagemagick` 和 `w3m` 是 neofetch 的依赖，要单独安装，否则无法正常显示图片。

### 更多 fetch

如果希望发现更多关于 fetch 的资源，查看 [awesome-fetch](https://github.com/beucismis/awesome-fetch)。没错，fetch 这个系列也有一个 awesome 系列。

## 开始正文

让我们开始正式介绍 fetch 的使用方法。

今天呢，主要介绍三个 fetch， 它们分别是 pfetch 、neofetch 和 fastfetch，有的追求简洁、有的强大但复杂，有的高性能。

总之，它们各具特色。

## pfetch

pfetch 的目标是通过 sh 实现一个简洁的系统信息收集工具。它支持 Linux、Android、BSD、Windows、MacOS、Solaris 等系统。

### 使用

输入以下命令查看效果：

```zsh
pfetch
```

效果如下：

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-04.png" >}}

### 配置

如上图打印的信息中，分为两个部分，系统 Logo 和系统软硬件信息。pfetch 使用环境变量配置具体显示什么信息。

如只显示 ASCII Logo，设置环境变量：

```
export PF_INFO="ascii"
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-05.png" >}}

如不显示 ASCII Logo，设置环境变量：

```zsh
export PF_INFO="title os host kernel uptime pkgs memory"
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-06.png" >}}

或者修改默认的 ASCII Logo 为 Linux：

```zsh
export PF_ASCII="linux"
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-07.png" >}}

更多配置项，自行查看 [pfetch 仓库文档](https://github.com/dylanaraps/pfetch)。

## neofetch

neofetch 是一款用 bash script 实现，快速且高度可定制的 fetch。它的功能比 pfecth 强大的多，支持展示 ASCII 和 图片，在 Linux、BSD、Mac OS X 和 Window 等系统上可运行。

不得不说，它的 github star 数量也是所有 fetch 中最多的。大部分人一般用的都是它吧。

要说它的缺点，一是性能低，因为是 bash script 实现，性能不高；二是已停止维护，好像 bug 已停止修复。

### 使用

我将以 Mac OS X 演示其使用。

简单实用，直接输入以下命令：

```
> neofetch
```
 
效果如下：

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-03.png" >}}

如上所示，和 pfetch 一样，两部分组成，一部分显示系统 Logo（Mac OS X），另一部分显示系统摘要信息。

### 配置

neofetch 功能强大，高度可定制，可通过命令选项或者配置文件进行定制，它的默认配置文件位于 `~/.config/neofetch/config.conf` 中。

不显示 ASCII Logo。

```bash
image_backend="off" # 或 nefetch --off
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-08.png" >}}

如显示字段定制：

```bash
print_info() {
    info title
    info underline

    info "OS" distro
    info "Host" model
    info "Kernel" kernel
    info "Uptime" uptime
    info "Packages" packages
    info "Shell" shell
    # info "Resolution" resolution
    # info "DE" de
    # info "WM" wm
    # info "WM Theme" wm_theme
    # info "Theme" theme
    # info "Icons" icons
    # info "Terminal" term
    # info "Terminal Font" term_font
    # info "CPU" cpu
    # info "GPU" gpu
    info "Memory" memory

    info cols
}
```

注释掉如上任一字段裁剪显示的字段。

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-09.png" >}}

或替换默认的 ASCII art 为其他 Logo，如替换为 ubuntu Logo，虽然，这么做不太合适。

```bash
ascii_distro="ubuntu" # 或 neofetch --ascii=ubuntu
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-10.png" >}}

同样，也可以将上篇文中的 `cowsay` 配置为 ASCII 部分的显示内容。

```bash
image_source="$(fortune | cowsay -W 40)" # 或 neofetch --source "$(fortune | cowsay)"
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-11.png" >}}

neofetch 还支持将左侧的内容从 ASCII art 替换为图片。

在 iterm2 的设置方式：

```bash
image_backend="iterm2"
image_source="$HOME/Pictures/avatar-transparency.png" # 或 neofetch --iterm2 ~/Pictures/avatar-transparency.png
```

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-13.png" >}}

一个问题，neofetch 有些场景下会无缘无故打印很多空行，要通过命令选项 `--size` 或配置参数 `image_size` 实现图片大小固定，同时再利用 `yoffset` 和 `gap` 调整出一个比较好看的效果。

如下所示：

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-14.png" >}}

neofetch 有一个问题，因为使用 bash script 实现，性能一般，明显能感觉到 info 打印时的卡顿。我们可通过 `info "OS" distro &` 的形式调用，即 `&` 实现异步执行，再利用 `wait` 等待，提升性能。

## fastfetch

和 pfetch、neofetch 不同，fastfetch 是 C 实现，自然地它的性能比之前两者就高上很多。

```bash
brew install fastfetch
```



