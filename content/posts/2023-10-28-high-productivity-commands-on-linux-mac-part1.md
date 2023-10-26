---
title: "我的终端环境：提升效率的 shell 命令（一）- 高效 ls 与 cd"
date: 2023-10-28T15:01:19+08:00
draft: true
comment: true
description: "本篇是提升效率的 Shell 命令个效率提升命令第一篇，介绍如何使用 exa 和 zoxide 替换 ls 和 cd。"
---

类 Unix 系统发展多年，不少老古董命令依然据着我们终端的绝大部分时间，而使用体验上却还是那么差强人意。

从本文开始，我将用四篇文章介绍如何提升终端效率的一系列命令，这些命令更具现代风格，希望能让你眼前一亮。

介绍介绍的命令，如下所示：

- exa 或 eza，替换默认的 ls 命令；
- zoxide，更智能地进行目录跳转，可替换默认的 cd 命令；
- fd，目录与文件搜索命令，比默认 find 更易于使用；
- ripgrep，与 grep 类似，用于搜索文件内容；
- fuzzy，模糊搜索器；
- entr，监听文件变化并执行相应的命令；
- httpie，更加人性化的进行 http 请求；
- jq，用于 JSON 数据的命令；

本文是系列第一篇，将会先介绍前两个命令，即 exa（eza） 和 zoxide。

## 前言

正式开始前，先推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令，可用于替换一大波一直在用的老古董命令。

[exa](https://the.exa.website/)
- 说明：可用于替换默认 ls 命令，这款在平时工作中使用最多的命令，exa 提供了更加丰富的特性。
- 安装：brew install exa
- 使用：
  - exa
  - exa --icons
  - exa -alh --git 或 exa --all --long --header --git
  - exa --tree
- 别名：
  - alias ls="exa --icons"
  - alias ll="exa --icons --long --header"
  - alias lg="exa --icons --long --header --git"
  - alias tree="exa --tree"
- 注：exa 已经停止维护，可使用 eza，基于 exa。

[zoxide](https://github.com/ajeetdsouza/zoxide)
- 说明：记录访问过的历史目录，以最少按键实现目录跳转，[zoxide vs zsh-z](https://www.libhunt.com/compare-zsh-z-vs-zoxide)
- 安装：brew install zoxide
- 配置：echo 'eval "$(zoxide init zsh --cmd cd)"' >> ~/.zshrc
- 使用：
  - cd ~/Code/golang-examples/
  - cd golang
  - cd ~/Code/golang-microservices
  - cdi golang
  - cd golang + <space+tab>


