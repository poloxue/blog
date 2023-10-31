---
title: "我的终端环境：高效 shell 命令（三）之 WEB 开发命令 - entr httpie jq"
date: 2023-11-02T17:43:02+08:00
draft: true
comment: true
description: ""
---

本文将介绍三个在日常的 Web 开发中能帮助提效的命令，它们分别是 entr、httpie、jq。

介绍说明如下：

- [entr](https://github.com/eradman/entr)，监听文件变化并执行相应的命令；
- [httpie](https://github.com/httpie/cli)，更加人性化的进行 http 请求；
- [jq](https://github.com/jqlang/jq)，用于处理 JSON 数据的命令；

下面开始一个个的介绍如何使用它们。

## entry

entry 命令的是什么？直接看演示效果，如下所示：

[entr][https://github.com/eradman/entr] 主要是用于监听文件变化事件并运行指定命令。

安装：brew install entr
- 使用：
  - 文件变化：
    - ls test.txt | entr echo "test.txt changed!"
  - 自动构建：
    - fd *.go | entr -r go run *.go 重启服务
  - 自动清屏：
    - -c 选项自动清屏；
    - fd *.go | entr -r go run *.go 重启服务
  - 新文件检测：
    - 循环检测新文件；
    - whle true do fd *.go | entr -d -r go run *.go; done
  - 循环检测退出
    - trap "commands you want to execute" SIGINT

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

