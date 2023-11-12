---
title: "我的终端环境：配置终端 Welcome 消息-纯粹装逼术"
date: 2023-11-15T19:38:48+08:00
draft: true
comment: true
description: "本文介绍如何配置终端的 welcome 消息，让你的终端看起来焕然一新。"
---


- 欢迎信息
  - /etc/motd - 登录欢迎文字；
  - 设置欢迎脚本；
- ascii 图片
  - ascii 文字：
    - figlet
    - https://www.patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20
  - ascii 图片：
    - http://ascii-art.de/ascii/
    - https://gopherproxy.meulie.net/gopher.viste.fr/1/attic/ascii-art
- 其他命令
  - cowsay
  - fortune
  - lolcat
- 一个命令
  - npx ascii-themes generate dev.to
- neofetch
  - 安装
    ```
    brew install neofetch
    ```
  - 使用 neofetch
- 
- fastfetch

https $(https ://api.github.com/repos/poloxue/images/branches/main | jq -r '.commit.commit.tree.url') | jq '.tree[].path'

