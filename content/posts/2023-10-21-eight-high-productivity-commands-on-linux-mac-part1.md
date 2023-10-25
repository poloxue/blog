---
title: "我的终端环境：Mac 和 Linux 下 7 个效率提升的命令（上）"
date: 2023-10-21T19:01:19+08:00
draft: true
comment: true
description: "本文介绍两个 ls 命令的替代品 - exa 和 colorls。"
---

类 Unix 系统发展多年，不少老古董命令还在占据着我们的终端，而它们的体验依旧差强人意。我将用两篇文章介绍 MacOS 和 Linux 下的 7 个可提升效率的命令，更具现代风格，希望能让你眼前一亮。

介绍介绍的命令，如下所示：

- exa 或 eza，替换默认的 ls 命令；
- zoxide，更智能地进行目录跳转，可替换默认的 cd 命令；
- fd，目录与文件搜索命令，比默认 find 更易于使用；
- ripgrep，与 grep 类似，用于搜索文件内容；
- fuzzy，模糊搜索；
- entr，监听文件变化并执行相应的命令；
- httpie，更加人性化的进行 http 请求；

本文是第一篇，将会先介绍前两个命令，即 exa 和 zoxide。

## 前言

正式开始前，推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令，可用于替换一直在用的一大波老古董命令。

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

本文介绍三个常见的 Shell 命令的替代品，希望给无聊的命令体验再增趣味。

