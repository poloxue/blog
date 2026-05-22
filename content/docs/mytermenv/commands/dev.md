---
date: 2024-05-05
title: "开发调试"
weight: 3
description: ""
---

# 开发调试

本文将介绍的 3 个命令，用于提高 Web 开发人员的日常工作效率。

对 Web 开发而言，除了基本的框架外，日常开发过程中还常用的就是调试工具。本文将要介绍的三个命令分别是 entr、httpie、jq。

- **[entr](https://github.com/eradman/entr)**，监听文件变化后执行相应命令，实现热重启；
- **[httpie](https://github.com/httpie/cli)**，体验更加友好的 HTTP 客户端命令；
- **[jq](https://github.com/jqlang/jq)**，强大的 JSON 数据解析命令，甚至可简单编程；

---

## entr

[entr](https://github.com/eradman/entr) 是一款能轻松实现简单热重启的小工具，具有语言和框架无关性。

特别说明，这个工具主要是用在开发调试阶段，不支持复杂的热重启能力。

### 安装

```bash
brew install entr
```

### 简单使用

直接通过演示观察它的行为：

```bash
$ ls text.txt | entr echo "file changed"
```

我们通过 `ls text.txt` 告诉 entry 监听的文件。当编辑并保存文件后，它会执行后续的命令 echo 打印提示。

我们只要对它稍做修改，就可以实现在监听到文件变化后，自动执行 `停止服务 → 重新编译 → 启动服务` 等一系列动作。

### 实现热重启

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

为这个服务加上热重启能力：

```bash
fd -e go | entr -r go run *.go
```

我们通过 `fd` 遍历所有 go 源码文件，当发现文件更新后会重新执行 `go run *.go`。`-r` 实现 reload，它会发送停止信号给常驻服务，让其重新运行。如果希望每次重启后清屏日志，加上 `-c` 选项。

### 支持新建文件

但还有个缺点：当创建新文件时，entry 默认无法检测。可以通过 `-d` 选项解决，它会检测新文件后停止 entr。由于是停止重新执行，要用 while 循环包装：

```bash
while true do fd *.go | entr -d -r go run *.go; done
```

但这样做还有个问题：用 Ctrl+C 强制退出后，while 循环还会重启。需要引入 `trap` 捕获退出信号：

```bash
#!/bin/bash
trap "exit;" SIGINT;

while true; do
  fd -e go | entr -rcd go run *.go;
done
```

把以上内容保存为 `run.sh`，在源码根目录下运行。现在你的服务就有了实时构建编译和简单的热重启能力。

### 限制

虽然 entr 能实现简单的热重启机制，但它不具备复杂的状态管理或零停机更新的能力。对于需要更高级热更新的场景，可能需要专门的工具或框架。

---

## httpie

[HTTPie](https://github.com/httpie/cli) 是一款注重用户体验的 HTTP 客户端命令，比 curl 更易于使用。

### 一个趣事

2022 年，HTTPie 的作者误操作将仓库设为私有，导致 54k star 直接清零。两年后重新恢复到 36k star，官方还写了一篇博客反思：[How We Lost 54k stars](https://httpie.io/blog/stardust)。

### 安装

```bash
brew install httpie
```

安装后有两个命令可用：`http` 和 `https`。`httpie` 命令用于管理自身的扩展和插件。

### 基础使用

```bash
# GET 请求
http GET http://httpbin.org/get name==poloxue age==18

# POST 请求
http POST http://httpbin.org/post name=poloxue age=18
```

### 易用设计

httpie 的命令行语法直观易懂。METHOD 在大部分情况下可以省略：

- 当有 data 时 → POST，如 `http ://httpbin.org/post name=poloxue`
- 当无 data 时 → GET，如 `http ://httpbin.org/get`

参数设置规则也很清晰：

- header：使用 `key:value` 设置
- query string：使用 `key==value`
- body json data：使用 `key=value` 即可

```bash
# 一次性设置 header、query 和 body
http --offline POST http://httpbin.org/post \
  X-API-Token:123456 \
  User-Agent:foo \
  name==poloxue \
  age==18 \
  email=poloxue123@gmail.com
```

`--offline` 是调试神器，只打印 HTTP 请求文本，不真正发出网络请求。

### 输出美化

HTTPie 默认格式化和高亮响应内容，成功、重定向和错误都会以不同颜色显示。

```bash
# 切换配色风格
http --style autumn GET ://httpbin.org/get
```

### 文件下载

```bash
http --download https://github.com/httpie/cli/archive/master.tar.gz
http --download -o httpie.tar.gz https://github.com/httpie/cli/archive/master.tar.gz
```

`-c` 选项启用断点续传。

---

## jq

[jq](https://github.com/jqlang/jq) 是一款强大的命令行 JSON 处理器。它让你在终端中对 JSON 数据进行过滤、映射、转换和格式化。在微服务和 API 主导的开发时代，jq 几乎是后端开发的标配工具。

### 安装

```bash
# macOS
brew install jq

# Debian/Ubuntu
sudo apt install jq

# 或直接下载单二进制文件（无依赖）
curl -L https://github.com/jqlang/jq/releases/download/jq-1.7/jq-linux-amd64 -o /usr/local/bin/jq
chmod +x /usr/local/bin/jq
```

### 格式化 JSON

这可能是 jq 最常用的场景：

```bash
# 把一行 JSON 格式化为可读格式
echo '{"name":"poloxue","age":18}' | jq .

# 配合 curl 使用
curl -s https://api.github.com/users/poloxue | jq .
```

### 选择字段

```bash
# 只提取需要的字段
curl -s https://api.github.com/repos/jqlang/jq \
  | jq '{name: .name, stars: .stargazers_count, license: .license.spdx_id}'
```

### 数组处理

```bash
# 列出所有 commits 的作者名
curl -s https://api.github.com/repos/jqlang/jq/commits \
  | jq '.[].commit.author.name'

# 筛选条件
curl -s https://api.github.com/repos/jqlang/jq/issues \
  | jq '.[] | select(.state == "open") | {title, number}'
```

### 嵌套结构

```bash
# 展开嵌套字段
echo '{"user":{"name":"alice","address":{"city":"beijing","zip":"100000"}}}' \
  | jq '.user.address.city'
# → "beijing"
```

### TSV 输出

```bash
# 以表格形式输出，方便粘贴到 Excel
curl -s https://api.github.com/repos/jqlang/jq/issues \
  | jq -r '.[] | [.number, .title, .state, .user.login] | @tsv'
```

### 编辑 JSON 文件

jq 不仅能读，还能写：

```bash
# 添加新字段
cat config.json | jq '. + {"version": "2.0.0"}' > config_new.json

# 更新字段
cat config.json | jq '.timeout = 30' > config_new.json

# 删除字段
cat config.json | jq 'del(.debug)' > config_new.json

# 原地编辑（借助 sponge）
cat config.json | jq '.timeout = 30' | sponge config.json
```

### 实战案例

从 CI 日志中提取测试失败的用例：

```bash
cat test-results.json | jq -r '
  .[] | select(.status == "failed") | "❌ \(.name) (\(.duration)s)"
'
```

### 配置别名

```bash
# ~/.zshrc
alias jqp='jq -r'       # 原始字符串输出
alias jqc='jq -c'       # 紧凑输出
alias json='jq .'       # 快速格式化 JSON
```
