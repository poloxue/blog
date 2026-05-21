---
title: "HTTPie - 人性化 HTTP 客户端"
weight: 2
---

# 人性化 HTTP Client 命令 - httpie

作为程序员，如果你要快速调试 HTTP 接口，首先想到的工具是什么？GUI 版 HTTP 客户端 Postman？命令行 curl？还是使用浏览器？不知道你是否抱怨过，如 curl 的很多传统命令基本都存在一个问题：虽然功能强大，但用户体验极差，基本不考虑使用者的感受。

本文介绍一款意外丢失 54k star，不到两年重获 34k 的 HTTP 客户端命令 - [HTTPie](https://github.com/httpie/cli/) ，它比 curl 更易于使用，是一款注重用户体验的 HTTP 客户端命令。

## 一个趣事

先说关于 HTTPie 的 star 数的一个趣事：

2022 年，由于 HTTPie 的作者出现了一次误操作，将仓位设为私有，导致 HTTPie 的 star 直接清零。经历了两年时间，如今重新恢复到 36k star。

当时，HTTPie 官方还写了一篇博客，专门反思了这次的事故，查看博客：[How We Lost 54k stars](https://httpie.io/blog/stardust)。

## 为什么推荐 HTTPie

HTTPie 的哲学是使 API 测试和调试变得直观友好，相比传统命令行工具（如 curl 和 wget），它提供了更丰富的功能和更易读的输出。

来自官方的示例：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-10-httpie-02.gif)

首先是，它的命令行语法直观易懂，如发送 GET 请求只需 `http GET <URL>`，而 POST 请求则是 `http POST <URL> <body>`，这比 curl 的命令行参数更易于理解和记忆。

它默认支持 JSON，它会将 HTTP 请求的 Content-Type 设置为 application/json 并自动将命令行数据参数序列化为 JSON，这使得很轻松即可与 JSON 主导的 RESTful API 交互。

它默认支持输出的格式化和着色，这使得响应易于阅读和分析。成功的响应、重定向和错误都会以不同的颜色高亮显示。它还有其他高级输出选项，允许以多种格式查看响应，包括原始 HTTP 数据和仅正文。它支持输出重定向到文件或其他程序，为处理响应数据提供了灵活性。

它允许创建会话，这意味着用户可以在多个请求之间保持某些参数不变（如认证令牌），从而简化了需要认证的 API 交互。

其它还有如它提供了丰富的插件和扩展的支持，得到了开源社区的支持，一个 HTTP 客户端工具的 star 数足有 54k 之多（曾经）。

OK，让我们结束废话，开始它的使用演示吧。

## 安装

如下是 MacOS 的安装命令：

```bash
brew install httpie
```

成功安装 HTTPie 后，我们将有两个命令可供使用，分别是 `http` 和 `https`。注意，用于发起 HTTP 请求的命令不是 `httpie`，而它提供的另外两个命令，分别是 `http` 和 `https`，而 `httpie` 则是用于管理自身的扩展和插件。

## 使用

我们先介绍最简单的使用案例，快速发起 GET 和 POST 请求。

HTTPie 发起 GET 请求：

```
http GET http://httpbin.org/get name==poloxue age==18
```

HTTPie 发起 POST 请求：

```
http POST http://httpbin.org/post name=poloxue age=18
```

`http/https` 命令的完整语法，如下所示：

```zsh
http/https [flags] [METHOD] URL [ITEM [ITEM]]
```

## 易用性设计

除了 HTTPie 的命令语法设计，为提高 HTTPie 的易用性，作者在一些细节上也做了一些设计，如我们发起一个 HTTP 请求，拷贝 url 到终端，可直接通过在 scheme 后新增空格，回车即可：

例如，GET 方法请求 http://httpbin.org/get，如下所示：

```zsh
http ://httpbin.org/get
```

演示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-10-httpie-03.gif)

进一步，其中的 METHOD 默认也可省略，具体规则是：

- 当有 data：POST，如 `http ://httpbin.org/post name=poloxue age=18`
- 当无 data：GET，如 `http ://httpbin.org/get`

HTTPie 还支持一种 offline 模式，它是 debug 神器，只打印 HTTP 请求文本，但不真正发出网络请求。

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

现在，让我们使用 offline 模式快速了解 HTTPie 中的 header 设置，query 参数 还有 JSON 请求体等是如何使用的吧。

它们的设置规则，如下所示：

- header：使用 `key:value` 设置 header；
- query string，使用 `key==value`，表示 query params，快速拼接 query string；
- body json data，使用 `key=value` 即可，更复杂结构，可通过命令管道或文件导入；

一个命令演示 Header、查询参数 和 JSON 请求的设置方法。

```bash
http --offline POST http://httpbin.org/post \
                    X-API-Token:123456\
                    User-Agent:foo\
                    name==poloxue\
                    age==18\
                    password=123456\
                    email=poloxue123@gmail.com
```

通过 `--offline` 启用 offline 调试模式调试输出结果：

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

## 输出美化

开始说到，HTTPie 还有一个能力，它默认格式化和高亮响应内容，如此的话，我们就不需要在去找其他的 json 格式化命令了。

默认效果如下：
![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-10-httpie-04.png)

我们可通过 `--style` 选项它的配色风格。

```bash
http --style autumn GET ://httpbin.org/get
```

如果想查看所有配色，输入 `http --style` 查看：

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

HTTPie 的格式方式也是可配置的，选项 `--pretty`，支持配置项有 `all|colors|format|none`。

- all，默认值，即高亮又格式化；
- format，不高亮但格式化；
- colors，高亮但不格式化；
- none，即禁用高亮和格式化；

```bash
http --style=autumn ://httpbin.org/get
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-10-httpie-05.png)

## 文件下载

除了这些常见的功能，下载文件也可通过 HTTPie 实现，一个选项 `--donwload` 即可。它还支持锻炼续传。

使用命令：
```bash
http --download https://github.com/httpie/cli/archive/master.tar.gz
```

也可指定文件下载路径，只要通过 `--output/-o` 选项指定输出的文件名称即可。

```bash
http --donwload -o httpie.tar.gz https://github.com/httpie/cli/archive/master.tar.gz
```

断点续传的话，一个选项 `--continue/-c` 即可启用。

演示效果，我选择下载一个大文件，飞书的海外版 Mac 安装包，。

```zsh
http -dco lark.dmg https://sf16-va.larksuitecdn.com/obj/lark-artifact-storage/49f5b75a/Lark-darwin_x64-6.11.16-signed.dmg
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-11/2023-11-02-high-productivity-shell-commands-part3-08.gif)

HTTPie 就介绍这么多，其他还有 cookie、session，authentication 等请自行查阅文档 [httpie 文档](https://httpie.io/docs/cli/)。

## 总结

总体而言，HTTPie 是一个即强大又好用的 HTTPie 客户端，值得收藏到你的日常开发调试箱中。

最后，感谢阅读，希望本文对你所有帮助，能进一步提升你在终端上的效率。

