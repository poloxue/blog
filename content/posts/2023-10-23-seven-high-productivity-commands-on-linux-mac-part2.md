---
title: "我的终端环境：Mac 和 Linux 下 7 个效率提升的命令（下）"
date: 2023-10-23T20:13:53+08:00
draft: true
comment: true
---

[fzf](https://github.com/junegunn/fzf) - fuzzy finder 模糊匹配
- 说明：
- 安装：brew install fzf
- 用法：
  - fzf
  - vim `fzf`
- 别名:
  - alias cdg='cd_global() {cd $(fd --type directory $1 $2 | fzf)}; cd_global'

[entr][https://github.com/eradman/entr]
- 说明：用于监听文件改变事件，并运行任意命令。
- 安装：brew install entr
- 使用：
  - ls test.txt | entr echo "test.txt changed"
  - fd *.go | entr -r go run *.go 重启服务

[httpie](https://github.com/httpie/cli/)

- 说明：更人性化的 HTTP 命令行客户端。
- 安装：brew install http
- 使用：
  - http GET http://httpbin.org/get
  - http POST http://httpbin.org/post name=poloxue age=18

