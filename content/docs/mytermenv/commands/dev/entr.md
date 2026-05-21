---
title: "entr - 实现语言无关热重启"
weight: 1
---

# entr - 实现语言无关热重启 

hi

在开发类似于 web 或其他常驻服务时，我们在修改代码后，要手动重启才能更新服务。如果你不是这种情况，或许框架默认支持热重启或是你集成了其他工具。

本文将介绍一款工具，它能轻松实现简单的热重启，它具有语言和框架无关性，是一个通用小工具，它就是 [entry](https://github.com/eradman/entr)。

特别说明，这个工具主要是用在开发调试阶段，不支持复杂的热重启能力。

## 什么 entry

简单来说，它是一个可监听文件状态变化并执行特定动作的命令。让我们直接通过演示观察它的行为。

```bash
$ ls text.txt | entr echo "file changed"
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-11/2023-11-02-high-productivity-shell-commands-part3-01.gif)

我们通过 `ls text.txt` 告诉 entry 监听的文件。当编辑并保存文件后，它通过指定命令 echo 打印提示 `file changed`。

我们只要对它稍做修改，就可以实现在监听到文件变化后，自动执行 `停止服务 -> 重新编译 -> 启动服务` 等一系列动作。

## 安装

```bash
# mac 安装命令
brew install entr
```

## 实现热重启

首先，开发一个简单 Go server 服务，文件是 `main.go`，代码如下： 

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
		_, _ = w.Write([]byte("Hello World!"))
	})

	fmt.Println("Server is listening on :3000")
	http.ListenAndServe(":3000", nil)
}
```

为这个服务加上热重启能力，命令如下：

```bash
fd -e go | entr -r go run *.go 重启服务
```

我们通过 `fd` 遍历所有 go 源码文件，当发现文件更新后会重新执行 `go run *.go`。注意，上述命令中，我们使用了 entr 的 `-r` 实现 reload，它会发送停止信号给常驻服务，让其重新运行。

如果你希望每次重新启动后，执行清屏日志，也可使用 -c 选项。

## 创建新文件

但这还有一个缺点，当创建新文件，entry 默认无法检测。针对这个场景，entry 提供了一个选项 `-d`，它在检测新文件后，停止 entr 命令。由于是停止重新执行，我们要通过 while 循环才能重启服务。

命令如下：

```bash
while true do fd *.go | entr -d -r go run *.go; done
```

看起来已经足够使用，但它有个问题，无法退出服务。因为即使是强制 kill 掉 `go run` 进程，依然会循环重启。我们要再引入一个命令 - trap，捕捉到退出信号停止循环，退出程序。

```zsh
trap "echo 'command you want to execute'" SIGINT; while true do sleep 10; done
```

这里的效果是捕捉 SIGINT 信号，并打印 `echo "command you want to execute"`。


效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-11/2023-11-02-high-productivity-shell-commands-part3-04.gif)

如此的话，简单改造下前面的命令。

现在，在源码根目录下创建一个名为 `run.sh` 的文件，源码如下所示：


```bash
#!/bin/bash
trap "exit;" SIGINT;

while true; do
  fd -e go | entr -rcd go run *.go;
done
```

现在，我们通过 `run.sh` 启动服务即可，我们为服务添加了一个实时构建编译和简单的热重启能力。

## 限制和考虑

虽然 `entr` 能够实现一种简单的热重启机制，但它并不具备复杂的状态管理或零停机更新的能力。

对于需要更高级热更新或热重载功能的应用，可能需要使用更专门化的工具或框架来实现，这些工具或框架能够处理如状态迁移、版本兼容性等更复杂的场景。

## 最后

总体而言，不同语言或框架可能都有自己的管理工具，但通过 entr 命令，我们能以最简单的方式实现热重启这个需求。

希望本文对你有所帮助，感谢阅读。

