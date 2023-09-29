---
title: "从头构建 Go Web 框架（二）：中间件"
date: 2021-10-28T13:35:05+08:00
comment: true
draft: false
tags: ["Golang", "Web"]
---

> 本系列文章写于 2014 年，相较于 golang 极短的发展历程，这已经是古董级别的一篇文章了，但 web 框架思想概念依然有效。系统通过这个系列文章，能让大家都现有 Go Web 框架有更深的认识。

本文是 "构建属于自己的 Web 框架" 系列文章中的第二篇，将介绍中间件的最佳实践。

- [第 1 部分：简介](https://www.poloxue.com/posts/2021-10-23-build-your-own-webframework-in-golang)，[Build Your Own Web Framework In Go](https://www.nicolasmerouze.com/build-web-framework-golang)
- [第 2 部分：Go 中间件：最佳实践和示例](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2)，[Part 2: Middlewares in Go: Best practices and examples](https://nicolasmerouze.notion.site/Part-2-Middlewares-in-Go-Best-practices-and-examples-32f41ae0e21b435c86cf9dd38bf0ff65)
- [第 3 部分：中间件数据共享]()，[Part 3: Share Values Between Middlewares](https://www.nicolasmerouze.com/share-values-between-middlewares-context-golang)
- [第 4 部分：第三方路由]()，[Part 4: Guide to 3rd Party Routers in Golang](https://nicolasmerouze.notion.site/Part-4-Guide-to-3rd-Party-Routers-in-Go-8ddcca5c360b4539a601ae383c9d7e5d)
- [第 5 部分：使用 MongoDB 实现 JSON-API]()，[How to implement JSON-API standard in MongoDB and Go](https://nicolasmerouze.notion.site/Part-5-How-to-implement-JSON-API-standard-in-MongoDB-and-Go-a3daeead140846e4a6ae1e3c01b47f52)
- [附加福利：上传文件到 s3]()，[Bonus: File Upload REST API with Go and Amazon S3](https://nicolasmerouze.notion.site/Bonus-File-Upload-REST-API-with-Go-and-Amazon-S3-1130fbacad7442c5b0a7df9320d792b4)

在编写 Go Web 应用时，代码重复是大多数开发者将会遇到的第一个问题。

为什么呢？

原因在于，在处理 request 时，诸如记录请求、将应用程序错误转换为 HTTP 500 错误、验证用户等一些操作，这是每个处理程序都要执行的动作。

# 基础入门

首先，使用 net/http 包创建一个简单版本的 HTTP Server 应用。

代码如下：

```go
import (
    "net/http"
    "fmt"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

阅读以上代码，我们看出 `http.HandleFunc` 通过接受参数 `location pattern` 和 `handler`，实现特定路径与处理程序的匹配映射。`handler` 有 2 个参数，分别是 response writer 和 request，分别用于请求响应写入和请求信息读取。

除了通过 `http.HandleFunc` 指定处理函数的方式，实现 `http.Handler` 接口也可以帮助我们达成同样目的。

接口定义如下：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

示例代码如下：

```go
type handler struct {}

func (h handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome!")
}

func main() {
    http.Handle("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

一旦实现了 `http.Handler` 的 `ServeHTTP(http.ResponseWriter, *http.Request)` 方法， 就能被 Go muxer (http.Handle(pattern, handler) function) 使用。


## 添加日志

现在，我们希望通过增加一个简单的日志，记录处理每个请求所花费的时间。

代码如下：

```go
func indexHandler(w http.ResponseWriter, r *http.Request) {
    t1 := time.Now()
    fmt.Fprintf(w, "Welcome!")
    t2 := time.Now()
    log.Printf("[%s] %q %v\n", r.Method, r.URL.String(), t2.Sub(t1))
}

func main() {
    http.HandleFunc("/", indexHandler)
    http.ListenAndServe(":8080", nil)
}
```

输出内容如下：

```log
[GET] / 1.43ms
[GET] /about 1.98ms
```

继续，我们增加第二个 handler。毕竟，只有一个路由的应用程序并不多。

```go
func aboutHandler(w http.ResponseWriter, r *http.Request) {
    t1 := time.Now()
    fmt.Fprintf(w, "You are on the about page.")
    t2 := time.Now()
    log.Printf("[%s] %q %v\n", r.Method, r.URL.String(), t2.Sub(t1))
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
    t1 := time.Now()
    fmt.Fprintf(w, "Welcome!")
    t2 := time.Now()
    log.Printf("[%s] %q %v\n", r.Method, r.URL.String(), t2.Sub(t1))
}

func main() {
    http.HandleFunc("/about", aboutHandler)
    http.HandleFunc("/", indexHandler)
    http.ListenAndServe(":8080", nil)
}
```

代码重复！ 该如何解决呢？

我们可以创建一个带有闭包的函数。但是，当我们有多个这样的函数时，它就会变得像 Javascript 中的回调一样糟糕，我们并不想如此。

# 链接处理

我们希望有一种类似 Rack、Ring、Connect.js 等的中间件系统的解决方案，链接多个处理程序。标准库中已经有这种实现的示例：

- `http.StripPrefix(prefix, handler)`；
- `http.TimeoutHandler(handler, duration, message)`。

它们将 `handler` 作为参数传递，返回一个新的 `handler`。如下所示：

```go
loggingHandler(recoverHandler(indexHandler))
```

中间件类似于 `func (http.Handler) http.Handler`，我们传递一个 `handler` 并返回一个 `handler`。我们就可以用 http.Handle(pattern, handler) 得到目标的处理程序。

实现代码如下：

```go
func main() {
    http.Handle("/", loggingHandler(recoverHandler(indexHandler)))
    http.ListenAndServe(":8080", nil)
}
```

但如此还是很麻烦，一遍又一遍地重复堆栈。 有没有什么更优雅的方式实现呢？

# 通用包 alice

[alice](https://github.com/justinas/alice) 是一个非常短小精悍的包，它优雅地实现了 `handler` 的链接调用。通过它，我们可以创建一个通用的 `handler` 列表，便于我们重复使用。

```go
func main() {
    commonHandlers := alice.New(loggingHandler, recoverHandler)
    http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
    http.Handle("/", alice.New(commonHandlers, bodyParserHandler).ThenFunc(indexHandler))
    http.ListenAndServe(":8080", nil)
}
```

问题解决！

我们已经有了一个使用标准接口的中间件系统，alice 有 50 行代码，一个非常小的依赖。如果想了解 alice 的实现细节，可自行阅读 [alice 源码](https://github.com/justinas/alice)。

# 多个参数的 Handler

alice 这样的中间件系统中，我们仍然不能使用类似 `http.StripPrefix(prefix, handler)` 具有多个参数的 `handler`。因为，它不是 `func (http.Handler) http.Handler` 类型函数。

怎么办？

我们可以通过定义新的 `handler` 实现兼容效果，保证满足 `func (http.Handler) http.Handler`。

```go
func myStripPrefix(h http.Handler) http.Handler {
    return http.StripPrefix("/old", h)
}
```

现在，新的 handler 我们在 alice 中间件系统可以开始使用了。

# 再谈 logging middleware

通过 alice，我们有了更加优雅的方式实现代码重复的删除。我们无需重新定义的一个新的 http.Handler 接口，标准接口即可满足要求，这意味学习成本非常低，依赖更少。

实现代码如下：

```go
func loggingHandler(next http.Handler) http.Handler {
  fn := func(w http.ResponseWriter, r *http.Request) {
    t1 := time.Now()
    next.ServeHTTP(w, r)
    t2 := time.Now()
    log.Printf("[%s] %q %v\n", r.Method, r.URL.String(), t2.Sub(t1))
  }

  return http.HandlerFunc(fn)
}
```

最后，我们使用 alice 将 loggingHandler 与其他 `handler` 链接起来。

```go
func aboutHandler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "You are on the about page.")
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Welcome!")
}

func main() {
  commonHandlers := alice.New(loggingHandler)
  http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
  http.Handle("/", commonHandlers.ThenFunc(indexHandler))
  http.ListenAndServe(":8080", nil)
}
```

大功告成！

# 新中间件：panic recovery

另一个真正必要的功能：`panic recovery`。

当生产环境出现 `panic`，应用程序会被关闭。即使，我们有一个监控负责应用程序检测重启，也会不可避免的停机一小段时间。我们必须捕捉和记录 panic，并保持应用程序运行。

使用 Go 和中间件系统，这会变得非常容易。 只需要我们创建一个 `defer` 函数恢复 `panic`，响应 HTTP 500 错误并记录 panic 即可。

```go
func recoverHandler(next http.Handler) http.Handler {
  fn := func(w http.ResponseWriter, r *http.Request) {
    defer func() {
      if err := recover(); err != nil {
        log.Printf("panic: %+v", err)
        http.Error(w, http.StatusText(500), 500)
      }
    }()

    next.ServeHTTP(w, r)
  }

  return http.HandlerFunc(fn)
}
```

现在，将它追加到我们的中间件 stack 中。

```go
func main() {
  commonHandlers := alice.New(loggingHandler, recoverHandler)
  http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
  http.Handle("/", commonHandlers.ThenFunc(indexHandler))
  http.ListenAndServe(":8080", nil)
}
```

# 总结

我们已经了解，`func (http.Handler) http.Handler` 是一种非常简单的中间件定义方法，它提供了一切所需，`http.Handler` 是一个标准接口。

而通过链接方式实现中间件系统，已经是非常流行的方案，比如 Gorilla 和标准库本身。 我认为这是最惯用的方式。

我们已经编写了两个中间件：logging 和 panice recovery。

几乎每个框架中都在重写它们，尽管它们的功能几乎一样。大多数框架都有自己特定的 `handler` 定义，很难与其他中间件协同使用。接下来的一部分，我们将会了解到，在中间件之间共享值时，我们可能要更改一些内容。但它其实没有那么大的变化，我们没有理由重写已有的中间件。

博文地址：[从头构建 Go Web 框架（二）](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2/)，译：[Part2: Middlewares in Go: Best practices and examples](https://nicolasmerouze.notion.site/Part-2-Middlewares-in-Go-Best-practices-and-examples-32f41ae0e21b435c86cf9dd38bf0ff65)。
