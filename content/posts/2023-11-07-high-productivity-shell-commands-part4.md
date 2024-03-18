---
title: "我的终端环境：高效 shell 命令（四）之 20+1 个 modern-unix 命令"
date: 2023-11-06T15:40:41+08:00
draft: false
comment: true
description: "本文将继续介绍 11 个 modern-unix 命令，快速一览。"
tags: ["zsh"]
---

{{< video yt_id="d6YlNM6aMHo" bb_id="875737608" >}}

本篇文章是介绍 [modern-unix](https://github.com/ibraheemdev/modern-unix) 仓库剩余的 20 个命令的上篇，外加 1 比 modern-unix 中更易于使用的命令。

> 系列阅读：
>
> - [我的终端环境：iTerm2 的安装与体验](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [我的终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [我的终端环境：高效 shell 命令（一）之目录文件命令 - exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [我的终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [我的终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [我的终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [我的终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [我的终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...

## 命令集合

第一篇文章中推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令。例如，最常用的命令，如 ls、cd、grep、find 等等命令，这个仓库都提供了合适的替代命令。

针对我们日常工作最常用的命令，我已用了三篇文章，从不同场景角度出发，介绍了它们的使用，从而提升终端的使用效率。毫无疑问，这些命令更具现代风格。

除前面已经介绍的命令，本文将会极简的方式介绍下剩余的其他命令。

## 一键安装

一键安装剩余的 20 + 1 (lf) 个命令，如下所示：

```zsh
brew install lsd git-delta dust duf broot ag mcfly choose-rust sd cheat tldr bottom glances gtop hyperfine gping procs curlie xh dog lf
```

## lsd

[lsd](https://github.com/lsd-rs/lsd)，号称 "下一代 ls 命令"，算是对 GNU ls 的重写，且与 ls 兼容，和 exa 功能上类似。

```zsh
lsd --long --header --git
```

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-01.png" >}}

## delta

[delta](https://github.com/dandavison/delta)，可用于支持 git 、diff 和 git grep 的语法高亮和分屏对比；

与 diff 一起使用：

```
diff -u main1.go main2.go | delta
```

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-02.png" >}}

与 git diff 一起使用

```
git show
```

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-03.png" >}}

## dust

[dust](https://github.com/bootandy/dust) - 使用 rust 实现，du+rust = dust，更直观的 du 命令。默认行为，以找到最大文件为第一选择。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-04.png" >}}

## duf

[duf](https://github.com/muesli/duf) - 视觉体验更佳 df，可作为 df 的替代品，按类型分组展示。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-05.png" >}}

## broof

[broot](https://github.com/Canop/broot) - 终端文件浏览器，类似于 mac 的 finder 的终端版本。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-06.gif" >}}

我觉得，如果说到命令行文件浏览器，lf 体验更佳，是一个更不错的选择，比起 broot，支持的 vim 方式导航和搜索。有兴趣也可以了解下。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-07.gif" >}}

### ag

[ag](https://github.com/ggreer/the_silver_searcher) - 类似于 ack 的代码搜索工具，但搜索速度更快。其实，和 rg 有点类似，但做了个压测，性能没有 rg 优秀。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-10.png" >}}

### mcfly

[mcfly](https://github.com/cantino/mcfly) - mcfly 智能搜索引擎取代 CTRL-R 默认的搜索引擎，会考虑你的工作环境和历史命令等，通过一个小型网络进行优先级排序。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-08.gif" >}}

### choose

[choose](https://github.com/theryangeary/choose) - 快速且易于使用的 cut 命令。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-11.png" >}}

### sd

[sd](https://github.com/chmln/sd) - 更直观的 "选择替换" 命令，可用于替换 sed。

```bash
sd old new filename
```

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-09.png" >}}

### cheat

[cheat](https://github.com/cheat/cheat) - 是 unix 命令的备忘录，是一个命令行辅助工具。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-12.png" >}}

### tldr

[tldr](https://github.com/tldr-pages/tldr) - "too long, don't read"，和 cheat 类似，列出某个命令的常见使用案例。它是一个社区驱动的项目。

{{< image "2023-11/2023-11-07-high-productivity-shell-commands-part4-13.png" >}}

{{< video bb_id="920838475" yt_id="mKQGfOq6Dpw" >}}

## bottom

[bottom](https://github.com/ClementTsang/bottom) - 运行于终端的跨平台系统监视器，可视化。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-14.png" >}}

## gtop

[gtop](https://github.com/aksakalli/gtop) - 和 bottom 有点类似，系统监控面板。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-15.png" >}}

## glances

[glances](https://github.com/nicolargo/glances) - 可用于替代 top/htop，监控 GNU/Linux、BSD、Mac OS 和 windows 系统。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-16.png" >}}

## hyperfine

[hyerfine](https://github.com/sharkdp/hyperfine) - 压测工具，可同时压测多个命令。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-17.gif" >}}

## gping

[gping](https://github.com/orf/gping) - ping 的终端可视化版本，体验不错，值得一试；

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-18.gif" >}}

## procs

[procs](https://github.com/dalance/procs) - rust 编写的 ps 的替代版本。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-19.gif" >}}

## curlie 

[curlie](https://github.com/rs/curlie) - http 客户端，号称是，兼具 curl 的强大与 httpie 的易用性。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-20.png" >}}

## xh

[xh](https://github.com/ducaale/xh) - 兼顾 httpie 的易用性，同时注重高性能的 http 客户端。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-21.gif" >}}

## dog

[dog](https://github.com/ogham/dog) - 依旧是 rust 实现的 DNS 分析工具 dig 的替代版本。

{{< image "2023-11/2023-11-12-high-productivity-shell-commands-part4-22.png" >}}
