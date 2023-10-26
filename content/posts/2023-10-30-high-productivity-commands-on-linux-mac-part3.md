---
title: "我的终端环境：提升效率的 shell 命令（四）- WEB 开发"
date: 2023-11-02T17:43:02+08:00
draft: true
comment: true
description: ""
---

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

[jq](https://github.com/jqlang/jq)

- 说明：用于处理 JSON 数据的命令；
- 安装：brew install jq
- 使用：
  - 格式化：httpie https://coderwall.com/bobwilliams.json | jq

