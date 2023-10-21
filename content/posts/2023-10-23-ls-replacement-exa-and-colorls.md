---
title: "2023 10 23 Ls Replacement Exa and Colorls"
date: 2023-10-21T19:01:19+08:00
draft: true
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
