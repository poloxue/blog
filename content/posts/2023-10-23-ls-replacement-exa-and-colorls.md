---
title: "我的终端环境：ls 命令替代品 exa 和 colorls"
date: 2023-10-21T19:01:19+08:00
draft: true
description: "本文介绍两个 ls 命令的替代品 - exa 和 colorls。"
---


文件列表

- 说明：ls 的替代方案 exa 或 colorls。
- [exa](https://github.com/ogham/exa):
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
