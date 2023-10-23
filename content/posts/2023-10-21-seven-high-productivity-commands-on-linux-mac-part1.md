---
title: "我的终端环境：Mac 和 Linux 下 7 个效率提升的命令（上）"
date: 2023-10-21T19:01:19+08:00
draft: true
comment: true
description: "本文介绍两个 ls 命令的替代品 - exa 和 colorls。"
---


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
  - fd pattern
  - fd -e md git

[ripgrep](https://github.com/BurntSushi/ripgrep)
- 说明：用于
- 安装：
- 使用：
  - rg text
  - rg -i text
  - rg -e '[0-9]{2}:[0-9]{2}'
  - rg -glob 'code/golang/*.go' -i NewUser
  - rg -glob 'code/golang/**/*.go' -i NewUser

本文介绍三个常见的 Shell 命令的替代品，希望给无聊的命令体验再增趣味。

[exa](https://github.com/ogham/exa)，用于替代默认的 ls 命令。

如果说 shell 中哪一个命令是使用最频繁的命令，毫无意味是 ls 命令。

## 安装

- 说明：ls 的替代方案 exa 或 colorls。
  - 安装：brew install exa
- colorls 安装：sudo gem install colorls
- 替代 ls
  ```zsh
  if [ -x "$(command -v exa)" ]; then
    alias ls=exa --icons
  fi
  ````

- colorls
  ```zsh
  if [ -x "$(command -v colorls)" ]; then
    alias ls="colorls"
    alias la="colorls -al"
  fi
  ```

## colorls


