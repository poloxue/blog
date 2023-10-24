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

本文是第一篇，将会先介绍前 4 个命令。

## 前言

正式开始前，推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令，可用于替换我们每天在用的一大波老古董命令。

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

[fd](https://github.com/sharkdp/fd)
- 说明：一款用于文件查找的命令，可用于替换默认命令 find；
- 安装：brew install fd
- 使用：
  - 简单搜索
    - fd pattern
  - 正则查询
    - fd '.*install'
    - fd '[0-9]{4}-[0-9]{2}'
  - 通配符
    - fd -g '*install*'
  - 无匹配符
    - fd -F '*install*'
  - 指定类型
    - fd -e md git
    - fd -e md golang
    - fd -e md zsh
  - 隐藏文件
    - fd -H config
  - 查询范围包含 gitignore
    - fd -I main.o
    - fd --no-ignore-vcs
  - 大小写敏感
    - fd -i readme.md # insensitive
    - fd -s readme.md # sensitive
  - 详细信息
    - fd -l .go
  - 大小过滤
    - fd -S +1000k
  - 查询目录
    - fd --type/-t directory golang
  - 查询并执行命令
    - fd --type/-t file -x wc -l
    - fd --type/-t file -X vim
- 性能：
  - 使用 hyperfine 测试，与 find 进行性能对比

[ripgrep](https://github.com/BurntSushi/ripgrep)
- 说明：用于
- 安装：
- 使用：
  - 默认搜索当前目录
    - rg main
  - 搜索指定目录
    - rg main ~/Code/golang-examples
  - 搜索指定文件
    - rg main ~/Code/golang-examples/main.go
  - 正则搜索
    - rg -e '[0-9]{2}:[0-9]{2}'
  - 支持 gitignore
  - 搜索并替换
    - rg main ~/Code/golang-examples r main # 只替换输出，未修改文件
  - rg -glob 'code/golang/*.go' -i NewUser
  - rg -glob 'code/golang/**/*.go' -i NewUser

本文介绍三个常见的 Shell 命令的替代品，希望给无聊的命令体验再增趣味。

