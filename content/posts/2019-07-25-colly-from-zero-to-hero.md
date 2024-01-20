
---
title: "Colly 从入门到不放弃指南"
date: 2019-07-25T13:35:36+08:00
draft: false
comment: true
tags: ["Golang", "Crawler"]
---

平时比较喜欢逛逛问答平台，比如 stackvoerflow，最近想聚合下一些平台的技术问答，比如 stackoverflow。

要完成这个工作，肯定是离不开爬虫工具。于是，我就想着顺便抽时间研究了 Go 的一款爬虫框架 colly。

# 概要介绍

colly 是 Go 实现的比较有名的一款爬虫框架，而且 Go 在高并发和分布式场景的优势也正是爬虫技术所需要的。它的主要特点是轻量、快速，设计非常优雅，并且分布式的支持也非常简单，易于扩展。

本文将基于 colly 的官方文档，介绍 colly 的学习指南，以及我对 colly 的理解。

# 如何学习

爬虫最有名的框架应该就是 Python 的 scrapy，很多人最早接触的爬虫框架就是它，我也不例外。它的文档非常齐全，扩展组件也很丰富。当我们要设计一款爬虫框架时，常会参考它的设计。之前看到一些文章介绍 Go 中也有类似 scrapy 的实现。

相比而言，colly 的学习资料就少的可怜了。刚看到它的时候，我总会情不自禁想借鉴我的 scrapy 使用经验，但结果发现这种生搬硬套并不可行。

到此，我们自然地想到去找些文章阅读，但结果是 colly 相关文章确实有点少，能找到的基本都是官方提供的，而且看起来似乎不是那么完善。没办法，慢慢啃吧！官方的学习资料通常都会有三处，分别是文档、案例和源码。

今天，暂时先从官方文档角度吧！正文开始。

# 官方文档

[官方文档](http://go-colly.org/docs/)介绍着重使用方法，如果是有爬虫经验的朋友，扫完一遍文档很快。我花了点时间将官网文档的按自己的思路整理了一版。

主体内容不多，涉及[安装](http://go-colly.org/docs/introduction/install/)、[快速开始](http://go-colly.org/docs/introduction/start/)、[如何配置](http://go-colly.org/docs/introduction/configuration/)、[调试](http://go-colly.org/docs/introduction/configuration/)、[分布式爬虫](http://go-colly.org/docs/best_practices/distributed/)、[存储](http://go-colly.org/docs/best_practices/storage/)、[运用多收集器](http://go-colly.org/docs/best_practices/multi_collector/)、[配置优化](http://go-colly.org/docs/best_practices/crawling/)、[扩展](http://go-colly.org/docs/best_practices/extensions/)。

其中的每篇文档都很短小，甚至是少的基本都不用翻页滚动。

# [如何安装](http://go-colly.org/docs/introduction/install/)

colly 的安装和其他的 Go 库安装一样简单。如下：

```bash
go get -u github.com/gocolly/colly
```

 一行命令搞定。So easy!

# [快速开始](http://go-colly.org/docs/introduction/start/)

我们来通过一个 hello word 案例快速体验下 colly 的使用。步骤如下：

第一步，导入 colly。

```go
import "github.com/gocolly/colly"
```

第二步，创建 collector。

```go
c := colly.NewCollector()
```

 第三步，事件监听，通过 callback 执行事件处理。

```go
// Find and visit all links
c.OnHTML("a[href]", func(e *colly.HTMLElement) {
	link := e.Attr("href")
	// Print link
	fmt.Printf("Link found: %q -> %s\n", e.Text, link)
	// Visit link found on page
	// Only those links are visited which are in AllowedDomains
	c.Visit(e.Request.AbsoluteURL(link))
})

c.OnRequest(func(r *colly.Request) {
	fmt.Println("Visiting", r.URL)
})
```

我们顺便列举一下 colly 支持的事件类型，如下：

- OnRequest 请求执行之前调用
- OnResponse 响应返回之后调用
- OnHTML 监听执行 selector
- OnXML 监听执行 selector
- OnHTMLDetach，取消监听，参数为 selector 字符串
- OnXMLDetach，取消监听，参数为 selector 字符串
- OnScraped，完成抓取后执行，完成所有工作后执行
- OnError，错误回调

最后一步，c.Visit() 正式启动网页访问。

```go
c.Visit("http://go-colly.org/")
```

案例的完整代码在 colly 源码的 _example 目录下 [basic](https://github.com/gocolly/colly/blob/master/_examples/basic/basic.go) 中提供。

# [如何配置](http://go-colly.org/docs/introduction/configuration/)

colly 是一款配置灵活的框架，提供了大量的可供开发人员配置的选项。默认情况下，每个选项都提供了较优的默认值。

如下是采用默认创建的 collector。

```go
c := colly.NewCollector()
```

配置创建的 collector，比如设置 useragent 和允许重复访问。代码如下：

```go
c2 := colly.NewCollector(
	colly.UserAgent("xy"),
	colly.AllowURLRevisit(),
)
```

我们也可以创建后再改变配置。

```go
c2 := colly.NewCollector()
c2.UserAgent = "xy"
c2.AllowURLRevisit = true
```

collector 的配置可以在爬虫执行到任何阶段改变。一个经典的例子，通过随机改变 user-agent，可以帮助我们实现简单的反爬。

```go
const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

func RandomString() string {
	b := make([]byte, rand.Intn(10)+10)
	for i := range b {
		b[i] = letterBytes[rand.Intn(len(letterBytes))]
	}
	return string(b)
}

c := colly.NewCollector()

c.OnRequest(func(r *colly.Request) {
	r.Headers.Set("User-Agent", RandomString())
})
```

前面说过，collector 默认已经为我们选择了较优的配置，其实它们也可以通过环境变量改变。这样，我们就可以不用为了改变配置，每次都得重新编译了。环境变量配置是在 collector 初始化时生效，正式启动后，配置是可以被覆盖的。

支持的配置项，如下：

```go
ALLOWED_DOMAINS (字符串切片)，允许的域名，比如 []string{"segmentfault.com", "zhihu.com"}
CACHE_DIR (string) 缓存目录
DETECT_CHARSET (y/n) 是否检测响应编码
DISABLE_COOKIES (y/n) 禁止 cookies
DISALLOWED_DOMAINS (字符串切片)，禁止的域名，同 ALLOWED_DOMAINS 类型
IGNORE_ROBOTSTXT (y/n) 是否忽略 ROBOTS 协议
MAX_BODY_SIZE (int) 响应最大
MAX_DEPTH (int - 0 means infinite) 访问深度
PARSE_HTTP_ERROR_RESPONSE (y/n) 解析 HTTP 响应错误
USER_AGENT (string)
```

它们都是些非常容易理解的选项。

我们再来看看 HTTP 的配置，都是些常用的配置，比如代理、各种超时时间等。

```go
c := colly.NewCollector()
c.WithTransport(&http.Transport{
	Proxy: http.ProxyFromEnvironment,
	DialContext: (&net.Dialer{
		Timeout:   30 * time.Second,          // 超时时间
		KeepAlive: 30 * time.Second,          // keepAlive 超时时间
		DualStack: true,
	}).DialContext,
	MaxIdleConns:          100,               // 最大空闲连接数
	IdleConnTimeout:       90 * time.Second,  // 空闲连接超时
	TLSHandshakeTimeout:   10 * time.Second,  // TLS 握手超时
	ExpectContinueTimeout: 1 * time.Second,  
}
```

# [调试](http://go-colly.org/docs/best_practices/debugging/)

在用 scrapy 的时候，它提供了非常好用的 shell 帮助我们非常方便地实现 debug。但非常可惜 colly 中并没有类似功能，这里的 debugger 主要是指运行时的信息收集。

debugger 是一个接口，我们只要实现它其中的两个方法，就可完成运行时信息的收集。

```go
type Debugger interface {
    // Init initializes the backend
    Init() error
    // Event receives a new collector event.
    Event(e *Event)
}
```

源码中有个典型的案例，[LogDebugger](https://github.com/gocolly/colly/blob/master/debug/logdebugger.go)。我们只需提供相应的 io.Writer 类型变量，具体如何使用呢？

一个案例，如下：

```go
package main

import (
	"log"
	"os"

	"github.com/gocolly/colly"
	"github.com/gocolly/colly/debug"
)

func main() {
	writer, err := os.OpenFile("collector.log", os.O_RDWR|os.O_CREATE, 0666)
	if err != nil {
		panic(err)
	}

	c := colly.NewCollector(colly.Debugger(&debug.LogDebugger{Output: writer}), colly.MaxDepth(2))
	c.OnHTML("a[href]", func(e *colly.HTMLElement) {
		if err := e.Request.Visit(e.Attr("href")); err != nil {
			log.Printf("visit err: %v", err)
		}
	})

	if err := c.Visit("http://go-colly.org/"); err != nil {
		panic(err)
	}
}
```

运行完成，打开 collector.log 即可查看输出内容。

# [分布式](http://go-colly.org/docs/best_practices/distributed/)

分布式爬虫，可以从几个层面考虑，分别是代理层面、执行层面和存储层面。

## 代理层面

通过设置代理池，我们可以将下载任务分配给不同节点执行，有助于提供爬虫的网页下载速度。同时，这样还能有效降低因爬取速度太快而导致IP 被禁的可能性。

colly 实现代理 IP 的代码如下：

```go
package main

import (
	"github.com/gocolly/colly"
	"github.com/gocolly/colly/proxy"
)

func main() {
	c := colly.NewCollector()

	if p, err := proxy.RoundRobinProxySwitcher(
		"socks5://127.0.0.1:1337",
		"socks5://127.0.0.1:1338",
		"http://127.0.0.1:8080",
	); err == nil {
		c.SetProxyFunc(p)
	}
	// ...
}
```

proxy.RoundRobinProxySwitcher 是 colly 内置的通过轮询方式实现代理切换的函数。当然，我们也可以完全自定义。

比如，一个代理随机切换的案例，如下：

```go
var proxies []*url.URL = []*url.URL{
	&url.URL{Host: "127.0.0.1:8080"},
	&url.URL{Host: "127.0.0.1:8081"},
}

func randomProxySwitcher(_ *http.Request) (*url.URL, error) {
	return proxies[random.Intn(len(proxies))], nil
}

// ...
c.SetProxyFunc(randomProxySwitcher)
```

不过需要注意，此时的爬虫仍然是中心化的，任务只在一个节点上执行。

## 执行层面

这种方式通过将任务分配给不同的节点执行，实现真正意义的分布式。

如果实现分布式执行，首先需要面对一个问题，如何将任务分配给不同的节点，实现不同任务节点之间的协同工作呢？

首先，我们选择合适的通信方案。常见的通信协议有 HTTP、TCP，一种无状态的文本协议、一个是面向连接的协议。除此之外，还可选择的有种类丰富的 RPC 协议，比如 Jsonrpc、facebook 的 thrift、google 的 grpc 等。

文档提供了一个 HTTP 服务示例代码，负责接收请求与任务执行。如下：

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"

	"github.com/gocolly/colly"
)

type pageInfo struct {
	StatusCode int
	Links      map[string]int
}

func handler(w http.ResponseWriter, r *http.Request) {
	URL := r.URL.Query().Get("url")
	if URL == "" {
		log.Println("missing URL argument")
		return
	}
	log.Println("visiting", URL)

	c := colly.NewCollector()

	p := &pageInfo{Links: make(map[string]int)}

	// count links
	c.OnHTML("a[href]", func(e *colly.HTMLElement) {
		link := e.Request.AbsoluteURL(e.Attr("href"))
		if link != "" {
			p.Links[link]++
		}
	})

	// extract status code
	c.OnResponse(func(r *colly.Response) {
		log.Println("response received", r.StatusCode)
		p.StatusCode = r.StatusCode
	})
	c.OnError(func(r *colly.Response, err error) {
		log.Println("error:", r.StatusCode, err)
		p.StatusCode = r.StatusCode
	})

	c.Visit(URL)

	// dump results
	b, err := json.Marshal(p)
	if err != nil {
		log.Println("failed to serialize response:", err)
		return
	}
	w.Header().Add("Content-Type", "application/json")
	w.Write(b)
}

func main() {
	// example usage: curl -s 'http://127.0.0.1:7171/?url=http://go-colly.org/'
	addr := ":7171"

	http.HandleFunc("/", handler)

	log.Println("listening on", addr)
	log.Fatal(http.ListenAndServe(addr, nil))
}
```

这里并没有提供调度器的代码，不过实现不算复杂。任务完成后，服务会将相应的链接返回给调度器，调度器负责将新的任务发送给工作节点继续执行。

如果需要根据节点负载情况决定任务执行节点，还需要服务提供监控 API 获取节点性能数据帮助调度器决策。

## 存储层面

我们已经通过将任务分配到不同节点执行实现了分布式。但部分数据，比如 cookies、访问的 url 记录等，在节点之间需要共享。默认情况下，这些数据是保存内存中的，只能是每个 collector 独享一份数据。

我们可以通过将数据保存至 redis、mongo 等存储中，实现节点间的数据共享。colly 支持在任何存储间切换，只要相应存储实现 [colly/storage.Storage](https://godoc.org/github.com/gocolly/colly/storage#Storage) 接口中的方法。

其实，colly 已经内置了部分 storage 的实现，查看 [storage](http://go-colly.org/docs/best_practices/storage/)。下一节也会谈到这个话题。

# [存储](http://go-colly.org/docs/best_practices/storage/)

前面刚提过这个话题，我们具体看看 colly 已经支持的 storage 有哪些吧。

[InMemoryStorage](https://github.com/gocolly/colly/blob/master/storage/storage.go)，即内存，colly 的默认存储，我们可以通过 collector.SetStorage() 替换。

[RedisStorage](https://github.com/gocolly/redisstorage)，或许是因为 redis 在分布式场景下使用更多，官方提供了[使用案例](http://go-colly.org/docs/examples/redis_backend/)。

其他还有 [Sqlite3Storage](https://github.com/velebak/colly-sqlite3-storage) 和 [MongoStorage](https://github.com/zolamk/colly-mongo-storage)。

# [多收集器](http://go-colly.org/docs/best_practices/multi_collector/)

我们前面演示的爬虫都是比较简单的，处理逻辑都很类似。如果是一个复杂的爬虫，我们可以通过创建不同的 collector 负责不同任务的处理。 

如何理解这段话呢？举个例子吧。

如果大家写过一段时间爬虫，肯定遇到过父子页面抓取的问题，通常父页面的处理逻辑与子页面是不同的，并且通常父子页面间还有数据共享的需求。用过 scrapy 应该知道，scrapy 通过在 request 绑定回调函数实现不同页面的逻辑处理，而数据共享是通过在 request 上绑定数据实现将父页面数据传递给子页面。

研究之后，我们发现 scrapy 的这种方式 colly 并不支持。那该怎么做？这就是我们要解决的问题。

对于不同页面的处理逻辑，我们可以定义创建多个收集器，即 collector，不同 collector 负责处理不同的页面逻辑。

```go
c := colly.NewCollector(
	colly.UserAgent("myUserAgent"),
	colly.AllowedDomains("foo.com", "bar.com"),
)
// Custom User-Agent and allowed domains are cloned to c2
c2 := c.Clone()
```

通常情况下，父子页面的 collector 是相同的。上面的示例中，子页面的 collector c2 通过 clone，将父级 collector 的配置也都复制了下来。

而父子页面之间的数据传递，可以通过 Context 实现在不同 collector 之间传递。注意这个 Context 只是 colly 实现的数据共享的结构，并非 Go 标准库中的 Context。

```go
c.OnResponse(func(r *colly.Response) {
	r.Ctx.Put("Custom-header", r.Headers.Get("Custom-Header"))
	c2.Request("GET", "https://foo.com/", nil, r.Ctx, nil)
})
```

如此一来，我们在子页面中就可以通过 r.Ctx 获取到父级传入的数据了。关于这个场景，我们可以查看官方提供的案例 [coursera_courses](http://go-colly.org/docs/examples/coursera_courses/)。

# [配置优化](http://go-colly.org/docs/best_practices/crawling/)

colly 的默认配置针对是少量站点的优化配置。如果你是针对大量站点的抓取，还需要一些改进。

## 持久化存储

默认情况下，colly 中的 cookies 和 url 是保存在内存中，我们要换成可持久化的存储。前面介绍过，colly 已经实现一些常用的可持久化的存储组件。

## 启用异步加快任务执行

colly 默认会阻塞等待请求执行完成，这将会导致等待执行任务数越来越大。我们可以通过设置 collector 的 Async 选项为 true 实现异步处理，从而避免这个问题。如果采用这种方式，记住增加 c.Wait()，否则程序会立刻退出。

## 禁止或限制 KeepAlive 连接

colly 默认开启 KeepAlive 增加爬虫的抓取速度。但是，这对打开的文件描述符有要求，对于长时间运行的任务，进程非常容易就能达到最大描述符的限制。

禁止 HTTP 的 KeepAlive 的示例代码，如下。

```go
c := colly.NewCollector()
c.WithTransport(&http.Transport{
    DisableKeepAlives: true,
})
```

# [扩展](http://go-colly.org/docs/best_practices/extensions/)

colly 提供了一些扩展，主要与爬虫相关的常用功能，如 referer、random_user_agent、url_length_filter 等。源码路径在 [colly/extensions/](https://github.com/gocolly/colly/tree/master/extensions) 下。

通过一个示例了解它们的使用方法，如下：

```go
import (
    "log"

    "github.com/gocolly/colly"
    "github.com/gocolly/colly/extensions"
)

func main() {
    c := colly.NewCollector()
    visited := false

    extensions.RandomUserAgent(c)
    extensions.Referrer(c)

    c.OnResponse(func(r *colly.Response) {
        log.Println(string(r.Body))
        if !visited {
            visited = true
            r.Request.Visit("/get?q=2")
        }
    })

    c.Visit("http://httpbin.org/get")
}
```

只需将 collector 传入扩展函数中即可。这么简单就搞定了啊。

那么，我们能不能自己实现一个扩展呢？

在使用 scrapy 的时候，我们如果要实现一个扩展需要提前了解不少概念，仔细阅读它的文档。但 colly 在文档中压根也并没有相关说明啊。肿么办呢？看样子只能看源码了。

我们打开 referer 插件的源码，如下：

```go
package extensions

import (
	"github.com/gocolly/colly"
)

// Referer sets valid Referer HTTP header to requests.
// Warning: this extension works only if you use Request.Visit
// from callbacks instead of Collector.Visit.
func Referer(c *colly.Collector) {
	c.OnResponse(func(r *colly.Response) {
		r.Ctx.Put("_referer", r.Request.URL.String())
	})
	c.OnRequest(func(r *colly.Request) {
		if ref := r.Ctx.Get("_referer"); ref != "" {
			r.Headers.Set("Referer", ref)
		}
	})
}
```

在 collector 上增加一些事件回调就实现一个扩展。这么简单的源码，完全不用文档说明就可以实现一个自己的扩展了。 当然，如果仔细观察，我们会发现，其实它的思路和 scrapy 是类似的，都是通过扩展 request 和 response 的回调实现，而 colly 之所以如此简洁主要得益于它优雅的设计和 Go 简单的语法。

# 总结

读完 colly 的官方文档会发现，虽然它的文档简陋无比，但应该介绍的内容基本上都涉及到了。如果有部分未涉及的内容，我也在本文之中做了相关的补充。之前在使用 Go 的 [elastic](https://github.com/olivere/elastic) 包时，同样也是文档少的可怜，但简单读下源码，就能立刻明白了该如何去使用它。

或许这就是 Go 的大道至简吧。

最后，如果大家在使用 colly 时遇到什么问题，官方的 example 绝对是最佳实践，建议可以抽时间一读。

我的博文：[Colly 从入门到不放弃指南](https://www.poloxue.com/posts/2019-07-25-colly-from-zero-to-hero)。
