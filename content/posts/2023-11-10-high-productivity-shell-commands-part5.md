---
title: "我的终端环境：高效 shell 命令（五）- 现代 Unix 命令集"
date: 2023-11-06T15:40:41+08:00
draft: true
comment: true
description: "本文介绍一个更符合现代风格的 Unix 命令的集合"
---

本文介绍 [modern-unix]() 仓库剩余的其他所用命令。

> 类 Unix 系统发展多年，不少老古董命令依然占据着终端的大部分时间，大有经久不衰、经典永留存的架势。但不得不提，经过这么多年，这些老古董命令在体验上还是那么差强人意。

## 命令集合

第一篇文章中推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令。例如，最常用的命令，如 ls、cd、grep、find 等等命令，这个仓库都提供了合适的替代命令。

针对我们日常工作最常用的命令，我已用了三篇文章，从不同场景角度出发，介绍了它们的使用，从而提升终端的使用效率。毫无疑问，这些命令更具现代风格。

除前面已经介绍的命令，本文将会极简的方式介绍一下剩余的其他命令。

## 一键安装

一键安装剩余的 20 个命令，如下所示：

```zsh
brew install lsd git-delta dust duf broot ag mcfly choose sd cheat tldr bottom glances gtop hyperfine gping procs curlie xh dog
```

## 快速一览

让我们快速一览这个现代风格的命令集合吧！

### lsd

[lsd](https://github.com/lsd-rs/lsd)，号称 "下一代 ls 命令"，算是对 GNU ls 的重写，且与 ls 兼容。

```zsh
lsd --long --header --git
```

### delta

[delta](https://github.com/dandavison/delta)，可用于支持 git 、diff 和 grep 的语法高亮和分屏对比；

与 diff 一起使用：

```
diff -u main1.go main2.go | delta
```

与 git diff 一起使用

```
git show
```

### dust

[dust](https://github.com/bootandy/dust) - 使用 rust 实现的更加直观的 du 命令。

### duf

[duf](https://github.com/muesli/duf) - 视觉体验更佳 df。

### broof

[broot](https://github.com/Canop/broot) - 终端文件浏览器，类似于 mac 的 finder 的终端版本。

### ag

[ag](https://github.com/ggreer/the_silver_searcher) - 类似于 ack 的代码搜索工具，但搜索速度更快。

### mcfly

[mcfly](https://github.com/cantino/mcfly) - 

### choose

[choose](https://github.com/theryangeary/choose) - 

### sd

[sd](https://github.com/chmln/sd) - 

### cheat

[cheat](https://github.com/cheat/cheat) - 

### tldr

[tldr](https://github.com/tldr-pages/tldr) -

### bottom

[bottom](https://github.com/ClementTsang/bottom) - 

### glances

[glances](https://github.com/nicolargo/glances) - 

### gtop

[gtop](https://github.com/aksakalli/gtop) - 

### hyperfine

[hyerfine](https://github.com/sharkdp/hyperfine) - 

### gping

[gping](https://github.com/orf/gping) - 

### procs

[procs](https://github.com/dalance/procs) - 

### curlie

[curlie](https://github.com/rs/curlie) - 

### xh

[xh](https://github.com/ducaale/xh) - 

### dog

[dog](https://github.com/ogham/dog) - 

