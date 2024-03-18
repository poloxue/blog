---
title: "我的终端环境：高效 shell 命令（三）之提效日常开发 - entr httpie jq"
date: 2023-11-02T17:43:02+08:00
draft: false
comment: true
description: "本文将介绍的 3 命令，用于提高 Web 开发人员们的日常效率，它们分别是 entr、httpie 和 jq。"
tags: ["zsh"]
---

{{< youtube bb_id="748039783" yt_id="XLlYFat4chE" >}}

本文将介绍的 3 命令，用于提高 Web 开发人员们的日常工作效率。

> 系列阅读：
>
> - [我的终端环境：iTerm2 的安装与体验](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [我的终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [我的终端环境：高效 shell 命令（一）之目录文件命令 - exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [我的终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [我的终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [我的终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [我的终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [我的终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...

## 前言

对 Web 开发而言，除了基本的框架外，日常开发过程中，还常用的必然就是调试工具。本文将要介绍的三个命令分别是 entr、httpie、jq，变主要是为了这个目的而生的。

大概说明，如下所示：

- [entr](https://github.com/eradman/entr)，它的主要作用是在当监听文件变化后，执行相应的命令；
- [httpie](https://github.com/httpie/cli)，相对于 curl，一款体验更加友好的 http client 命令；
- [jq](https://github.com/jqlang/jq)，一款强大的 JSON 数据的解析命令，甚至可简单的编程；

这三个命令在日常的 web 开发过程中扮演着不同的角色。

开始具体介绍。

## entr 实现 Live Reloading

[entry](https://github.com/eradman/entr) 命令的是什么？直接看演示效果，如下所示：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-01.gif" >}}

大概看出，[entr](https://github.com/eradman/entr) 是用于监听文件变化并执行指定命令。看到这，有没有想到啥？对，entr 可用于服务的热加载（Live Reloading）。

### 安装

安装命令，如下所示：

```zsh
brew install entr
```

### 案例

最直接的示例效果，监听文件变化并发出通知，如下所示：

```zsh
ls test.txt | entr echo "test.txt changed!"
```

OK, 现在想到啥了吗？这个命令对于 Web 开发人员有什么样的价值呢？

回想一下，平时开发 web 程序时，我们是不是经常手动重启服务呢？

如果不是，或许是你所用框架默认支持或是你通过其他工具实现了 live-reading，比如我所有的博客工具 hugo，就有内置了自动加载的能力。

但由于这并不是框架默认能力，不同语言不同框架都要去寻找相应的实现方案。

庆幸的是，通过 `entr`，可以很方便地实现这个功能。

以一个 Go server 为例，文件名是 `main.go`，代码如下所示： 

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

通过如下命令，`fd` 实现：

```
fd -e go | entr -r go run *.go 重启服务
```

效果如下：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-02.gif" >}}

另外，如希望每次重新启动后，执行清屏操作清屏，可使用 -c 选项。效果如下所示：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-03.gif" >}}

但是，这里还有一个缺点，检查有新文件创建，entry 默认无法检测。

针对这个问题，entry 提供了一个选项 `-d`，它在检测到新文件后，会重新执行 entr 命令。由于是停止重新执行，要通过使用 while 循环执行。

完整命令，如下所示：

```bash
while true do fd *.go | entr -d -r go run *.go; done
```

看起来已经足够使用是吗？但其实，这里依然有个大问题，那就是 while 如果无法退出。即使是强制 kill 掉 `go run` 进程，依然重新启动。

引入一个命令 - trap，协助解决。它的主要作用是捕捉指定信号就会执行相应命令。

```zsh
trap "echo 'command you want to execute'" SIGINT; while true do sleep 10; done
```

捕捉 SIGINT 信号，并打印 `echo "command you want to execute"`。

注：这个命令建议不要尝试，不然只要杀死终端才能退出。

效果如下所示：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-04.gif" >}}

如此的话，简单改造下前面的命令。

现在，在源码根目录下创建一个名为 `run.sh` 的文件，源码如下所示：


```bash
#!/bin/bash
trap "exit;" SIGINT;

while true; do
  fd -e go | entr -rcd go run *.go;
done
```

查看演示效果，如下所示：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-05.gif" >}}
 
到这里，就为我们的项目添加了一个实时构建编译的能力。

虽说，不同语言可能都有自己的管理工具，现在有了 entr 这个命令，就能以最简单的方式实现我们的需求。

## httpie 人性化 HTTP Client

[httpie](https://github.com/httpie/cli/) 是一款更人性化的 HTTP 命令行客户端，简单来说，比 curl 更加易于使用。

### 安装

```zsh
brew install httpie
```

### 案例

最简单的使用案例，快速发起 GET 和 POST 请求。

GET 请求：
```
http GET http://httpbin.org/get name==poloxue age==18
```

POST 请求：
```
http POST http://httpbin.org/post name=poloxue age=18
```

http 命令的完整语法，如下所示：

```zsh
http [flags] [METHOD] URL [ITEM [ITEM]]
```

#### 一般使用 

从易用性角度，如果拷贝一个 url 可直接通过在 scheme 之后添加一个 \<space\> 便可直接使用：

例如，GET 方法请求 http://httpbin.org/get，如下所示：

```zsh
http ://httpbin.org/get
```

演示效果：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-06.gif" >}}

另外，METHOD 默认也可省略的，省略规则是：

- 当有 data：POST，如 `http ://httpbin.org/post name=poloxue age=18`
- 当无 data：GET，如 `http ://httpbin.org/get`

httpie 还支持一种 offline 模式，debug 神器，只打印 HTTP 请求文本，但不进行网络请求；

```bash
http --offline ://httpbin.org/get
```

输出内容：
```bash
GET /get HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: httpbin.org
User-Agent: HTTPie/3.2.2
```

现在，通过 offline 快速了解 httpie 的 header 设置，query 参数、JSON 请求体 等；

几个规则，如下所示：

- header：使用 `key:value` 设置 header；
- query string，使用 `key==value`，表示 query params，快速拼接 query string；
- body json data，使用 `key=value` 即可，更复杂结构，可通过命令管道或文件导入；

一个命令演示下如何设置：

```
http --offline POST http://httpbin.org/post \
                    X-API-Token:123456\
                    User-Agent:foo\
                    name==poloxue\
                    age==18\
                    password=123456\
                    email=poloxue123@gmail.com
```

启用 offline 调试模式，输出结果：

```bash
POST /post?name=poloxue&age=18 HTTP/1.1
Accept: application/json, */*;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Content-Length: 55
Content-Type: application/json
Host: httpbin.org
User-Agent: foo
X-API-Token: 123456

{
    "email": "poloxue123@gmail.com",
    "password": "123456"
}
```

#### 输出美化

httpie 还有一个能力，它默认会对返回数据进行格式化与语法高亮处理。如此一来，其他的 json 格式化命令就用不到了，如早前介绍的 `pp_json` 这个命令。毕竟，httpie 的格式美化能力更强大。

想修改它的配色风格？

通过 `--style` 选项即可修改。


```bash
http --style autumn GET ://httpbin.org/get
```

如果想查看所有可用该配色，直接输入 `http --style` 即可查看：

输出内容如下：

```bash
usage:
    http -s/--style {abap, algol, algol_nu, arduino, auto, autumn, borland, bw, colorful, default, dracula, emacs,
friendly, friendly_grayscale, fruity, github-dark, gruvbox-dark, gruvbox-light, igor, inkpot, lightbulb, lilypond,
lovelace, manni, material, monokai, murphy, native, nord, nord-darker, one-dark, paraiso-dark, paraiso-light,
pastie, perldoc, pie, pie-dark, pie-light, rainbow_dash, rrt, sas, solarized, solarized-dark, solarized-light,
staroffice, stata, stata-dark, stata-light, tango, trac, vim, vs, xcode, zenburn} [METHOD] URL [REQUEST_ITEM ...]

error:
    argument --style/-s: expected one argument

for more information:
    run 'http --help' or visit https://httpie.io/docs/cli
```

httpie 的格式方式也是可配置的，选项 `--pretty`，可配置值有 `all|colors|format|none`，看名字大概能猜出它们的区别：

- all，默认值，即高亮又格式化；
- format，不高亮但格式化；
- colors，高亮但不格式化；

```
http --style=autumn ://httpbin.org/get
```

演示效果：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-07.png" >}}

#### 文件下载

通过 `--donwload` 即可：

```zsh
http --download https://github.com/httpie/cli/archive/master.tar.gz
```

文件下载，除去最基本的需求，肯定还是要支持断点续传的。


要支持断点续传的话，首先，要通过 `--output/-o` 选项指定输出的文件名称。

```bash
http --donwload -o httpie.tar.gz https://github.com/httpie/cli/archive/master.tar.gz
```

断点续传，要启用 `--continue/-c` 选项。

为了掩饰效果，选择一个大文件进行下载，下载飞书的海外版 Mac 安装包。

命令如下：

```zsh
http -dco lark.dmg https://sf16-va.larksuitecdn.com/obj/lark-artifact-storage/49f5b75a/Lark-darwin_x64-6.11.16-signed.dmg
```

演示效果，如下所示：

{{< image "2023-11/2023-11-02-high-productivity-shell-commands-part3-08.gif" >}}

httpie 就介绍这么多，其他还有 cookie、session，authentication 等等请自行查阅文档 [httpie 文档](https://httpie.io/docs/cli/)。

## jq - 强大的 JSON 处理器

[jq](https://github.com/jqlang/jq)，一款可用于处理解析 JSON 文本的命令，它非常强大，甚至是可编程的。

### 安装

```bash
brew install jq
```

### 案例

最简单的使用场景，JSON 文本格式化，命令如下所示：

```
curl https://coderwall.com/bobwilliams.json | jq '.'
```

输出内容如下：

```
{
  "id": 26098,
  "username": "bobwilliams",
  "name": "Bob Williams",
  "location": "Charleston, SC",
  "karma": 1010,
  "accounts": {
    "github": "bobwilliams",
    "twitter": "_bobwilliams"
  },
  "about": "hacker, lifter, drummer and happily married, proud father of three\n\n\n[LinkedIn](http://www.linkedin.com/pub/bob-williams/17/a6a/314)\n\n",
  "title": "CTO",
  "company": "SPARC",
  "team": 4521,
  "thumbnail": "https://coderwall-assets-0.s3.amazonaws.com/uploads/user/avatar/26098/photo.JPG",
  "endorsements": 1010,
  "specialities": [],
  "badges": [
    {
      "name": "Platypus",
      "description": "Have at least one original repo where scala is the dominant language",
      "created_at": "2014-04-22T09:02:31.443Z",
      "badge": "https://dj1symwmxvldi.cloudfront.net/assets/badges/platypus-edcb8d16952f9cb27e9a1644d7d38b5606ff8f0c8a55869c4e0d8c42ed4ea637.png"
    },
    { ... }, { ... }, { ... }, { ... }, { ... },
  ]
}
```

如果是通过 httpie 发起请求，jq 的这个能力就显的有点鸡肋了，虽说文本来源不一定都是 HTTP，但它还是占了大头不是。

jq 的强大之处在于，它对 JSON 随心所欲的处理能力，如 object 字段访问，数组访问，重新构造对象与数组，甚至是支持统计、排序、过滤等，基于原始 JSON 生成一份更有价值的数据。

以一个需求为例，假设我希望将以上 JSON 进行一系列处理，得到如下的结果。

```json
{
  "name": "xxx",
  "title": "xxx",
  "company": "xxx",
  "badges": ["name01", "name02", "..."],
  "badge_count": 0
}
```

注：其中，badges 只要求取出按 created_at 排序最新的 5 个，导出它们的 name。

通过如下命令将 json 响应内容保存到一个 JSON 文件：

```
https -o bobwilliams.json https://coderwall.com/bobwilliams.json
```

如何导出部分字段？

```bash
 jq '{name: .name, title: .title, company: .company}' bobwilliams.json
```

输出如下：

```bash
{
  "name": "Bob Williams",
  "title": "CTO",
  "company": "SPARC"
}
```

如何导出 badges 最新的 5 个 badges:

```
jq '.badges | sort_by(.created_at) | reverse | [.[:5][].name]' bobwilliams.json
```

通过 sort_by 按 created_at 升序排列并通过 reverse 转为降序，使用 `.[:index]` 的语法进行切片，通过 `[]` 重新构造数组。

输出如下：

```bash
[
  "Platypus",
  "Python",
  "Narwhal",
  "Forked",
  "Charity"
]
```

那么，还剩最后一个问题，只展示了最新的 5 个 badges，本身一共有几个徽章呢？

```
jq '.badges | length' bobwilliams.json
```

输出如下：

```bash
11
```

最后，将以上各个部分进行重组，整合成一个新的 object 即可，完整表达式：

```bash
jq '{name, title, company, badges: [.badges | sort_by(.created_at) | reverse | .[:5][].name], badge_lenght: .badges | length}' bobwilliams.json
```

输出如下所示：

```json
{
  "name": "Bob Williams",
  "title": "CTO",
  "company": "SPARC",
  "badges": [
    "Platypus",
    "Python",
    "Narwhal",
    "Forked",
    "Charity"
  ],
  "badge_lenght": 11
}
```

jq 命令就是这么强大，如果平时在 Web 开发时，遇到大 JSON 输出，可通过 jq 帮助进行更加细致的格式化。


## 总结

本文介绍了三个开发阶段常用的命令，分别是用于实时更新重启的命令 entr，更人性化的 http 客户端 httpie 和一款强大的 JSON 数据处理命令 jq。

让它们提效我们的日常工作吧！
