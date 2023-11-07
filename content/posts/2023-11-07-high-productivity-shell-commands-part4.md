---
title: "我的终端环境：高效 shell 命令（四）- 11 个 modern-unix 命令"
date: 2023-11-06T15:40:41+08:00
draft: true
comment: true
description: "本文将继续介绍 11 个 modern-unix 命令，快速一览。"
tags: ["zsh"]
---

本篇文章是介绍 [modern-unix](https://github.com/ibraheemdev/modern-unix) 仓库剩余的 20 个命令的上篇。

## 命令集合

第一篇文章中推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令。例如，最常用的命令，如 ls、cd、grep、find 等等命令，这个仓库都提供了合适的替代命令。

针对我们日常工作最常用的命令，我已用了三篇文章，从不同场景角度出发，介绍了它们的使用，从而提升终端的使用效率。毫无疑问，这些命令更具现代风格。

除前面已经介绍的命令，本文将会极简的方式介绍下剩余的其他命令。

## 一键安装

一键安装剩余的 20 个命令，如下所示：

```zsh
brew install lsd git-delta dust duf broot ag mcfly choose-rust sd cheat tldr bottom glances gtop hyperfine gping procs curlie xh dog
```

## lsd

[lsd](https://github.com/lsd-rs/lsd)，号称 "下一代 ls 命令"，算是对 GNU ls 的重写，且与 ls 兼容，和 exa 功能上类似。

```zsh
lsd --long --header --git
```

{{< image "./2023-11-07-high-productivity-shell-commands-part4-01.png" >}}

## delta

[delta](https://github.com/dandavison/delta)，可用于支持 git 、diff 和 grep 的语法高亮和分屏对比；

与 diff 一起使用：

```
diff -u main1.go main2.go | delta
```

{{< image "./2023-11-07-high-productivity-shell-commands-part4-02.png" >}}

与 git diff 一起使用

```
git show
```

{{< image "./2023-11-07-high-productivity-shell-commands-part4-03.png" >}}

## dust

[dust](https://github.com/bootandy/dust) - 使用 rust 实现，du+rust = dust，更直观的 du 命令。默认行为，以找到最大文件为第一选择。

{{< image "./2023-11-07-high-productivity-shell-commands-part4-04.png" >}}

## duf

[duf](https://github.com/muesli/duf) - 视觉体验更佳 df，可作为 df 的替代品，按类型分组展示。

{{< image "./2023-11-07-high-productivity-shell-commands-part4-05.png" >}}

## broof

[broot](https://github.com/Canop/broot) - 终端文件浏览器，类似于 mac 的 finder 的终端版本。

{{< image "./2023-11-07-high-productivity-shell-commands-part4-06.gif" >}}

不过，要说命令行文件浏览器， lf 似乎是一个更不错的选择。

### ag

[ag](https://github.com/ggreer/the_silver_searcher) - 类似于 ack 的代码搜索工具，但搜索速度更快。

### mcfly

[mcfly](https://github.com/cantino/mcfly) - 替换 CTRL-R 的默认搜索能力，支持历史记录菜单列表的效果。

### choose

[choose](https://github.com/theryangeary/choose) - 快速且易于使用的 cut 命令。

### sd

[sd](https://github.com/chmln/sd) - 更直观的 "选择替换" 命令，可用于替换 sed。

### cheat

[cheat](https://github.com/cheat/cheat) - 是 unix 命令的备忘录，是一个命令行辅助工具。

### tldr

[tldr](https://github.com/tldr-pages/tldr) - "too long, don't read"，和 cheat 类似，列出某个命令的常见使用案例。它是一个社区驱动的项目。

