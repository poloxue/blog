---
title: "从头构建 Go Web 框架（四）：第三方路由集成"
date: 2023-10-09T13:35:05+08:00
draft: false
comment: true
tags: ["Golang", "Web"]
---

> 本系列文章写于 2014 年，相较于 golang 极短的发展历程，这已经是古董级别的一篇文章了，但 web 框架思想概念依然有效。希望通过翻译这个系列文章，能让大家都现有 Go Web 框架有更深的认识。

本文是 "构建属于自己的 Web 框架" 系列文章中的第四篇，将介绍如何在 Go 中使用三方路由。

- [第 1 部分：简介](https://www.poloxue.com/posts/2021-10-23-build-your-own-webframework-in-golang)，[Build Your Own Web Framework In Go](https://www.nicolasmerouze.com/build-web-framework-golang)
- [第 2 部分：Go 中间件：最佳实践和示例](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2)，[Part 2: Middlewares in Go: Best practices and examples](https://nicolasmerouze.notion.site/Part-2-Middlewares-in-Go-Best-practices-and-examples-32f41ae0e21b435c86cf9dd38bf0ff65)
- [第 3 部分：中间件数据共享](https://www.poloxue.com/posts/2023-09-30-build-your-own-webframework-in-golang-part-3)，[Part 3: Share Values Between Middlewares](https://nicolasmerouze.notion.site/Part-3-Share-Values-Between-Middlewares-dca6f9448a0c4be68b0da30137c38875)
- [第 4 部分：第三方路由](https://www.poloxue.com/posts/2023-10-09-build-your-own-webframework-in-golang-part-4)，[Part 4: Guide to 3rd Party Routers in Golang](https://nicolasmerouze.notion.site/Part-4-Guide-to-3rd-Party-Routers-in-Go-8ddcca5c360b4539a601ae383c9d7e5d)
- [第 5 部分：使用 MongoDB 实现 JSON-API]()，[How to implement JSON-API standard in MongoDB and Go](https://nicolasmerouze.notion.site/Part-5-How-to-implement-JSON-API-standard-in-MongoDB-and-Go-a3daeead140846e4a6ae1e3c01b47f52)

基于 Go 标准库 `net/http`，已经足够写出一个 Web 应用。但不足的是，它提供的路由能力 `http.Handle(pattern, handler)` 还是过于单一，只能实现一些静态路由。

这就是为什么我们需要一个优秀的三方路由。

然如此多的第三方路由，都有各自的特点，究竟该如何选择？

当接触一门新的编程语言，如果有 10 个不同库实现相同能力，将很难了解什么是最佳实践。我们希望有一种速度快、内存高效且易于使用的 router。

如下是我认为 Go 中最常用的 router，将从执行速度、内存消耗等维度对比。

## gorilla/mux

[gorilla/mux](https://github.com/gorilla/mux) 是一款成熟的 router，同时也是 Go 中最流行的三方路由。它有着丰富的功能，缺点是速度慢且内存消耗验证。

且，gorilla/mux 支持正则 URL 参数约束，如下所示：

```go
r := mux.NewRouter()
r.HandleFunc("/teas/{category}/", TeasCategoryHandler)
r.HandleFunc("/teas/{category}/{id:[0-9]+}", TeaHandler)
```

HTTP 方法配置路由，如下：

```go
r.Methods("GET", "HEAD").HandleFunc("/teas/{category}/", TeasCategoryHandler)
```

和其他路由的不同，gorilla/mux 有丰富的内置匹配规则，支持如 host（如子域名）、前缀、协议（http、https 等）、HTTP 头、查询参数。如果这些还不能满足你，通过自定义方式，如下方式：

```go
// Proto      string // "HTTP/1.0"
// ProtoMajor int    // 1
// ProtoMinor int    // 0
r.MatcherFunc(func(r *http.Request, rm *RouteMatch) bool {
  return r.ProtoMajor == 0
})
```

在 Handler 函数中，通过 `mux.Vars(request)` 可获取 URL 参数，它和上文介绍的 gorilla/context 类似。

代码如下：

```go
func myHandler(w http.ResponseWriter, r *http.Request) {
  vars := mux.Vars(r)
  category := vars["category"]
}
```

这个方案的优势是，它与 `http.Handler` 接口兼容。这点其实非常重要，因为我们的应用越多，共享 handler 和 middleware 的可能越大，就更加需要遵循一定的规则。

优势：功能强大，轻松创建复杂的路由规则，且与 `http.Handler` 兼容。
劣势：速度慢且内存消耗严重，如果看中速度的话，它不适合你。

## httprouter

httprouter, 号称 "最快的 router"。httprouter 的作者对不同的 router 做了基准测试，具体查看 [go-http-routing-benchmark](https://github.com/julienschmidt/go-http-routing-benchmark)。

httprouter 比 gorilla/mux 简单，但它不支持约束和正则，对于 REST API 而言，这个缺点的影响不大，但如果希望创建复杂的路由，这个简化设计就会大大限制它的适用范围。

还有，它与 `http.Handler` 不兼容，它定义了一个新的 interface，拥有三个参数，其中第三个参数用于访问 URL 参数。

示例代码：

```go
func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func main() {
    router := httprouter.New()
    router.GET("/", Index)
    router.GET("/hello/:name", Hello)
}
```

但这个问题也容易解决，将 URL 参数注入 context 中，实现在标准 interface `http.Handler` 和 httprouter 接口间的转换。这种方式会损失一些性能，但依然是一个 faster router。

如何实现？后续具体实现时介绍。

优点：快。

缺点：与 http.Handler 不兼容。

## Pat

[Pat](https://github.com/bmizerany/pat) 也是一个流行且简单的 router。它与 `http.Handler` 完全兼容。但它不是用 context 存储 URL 参数，而是将参数保存在 request 中，通过 `r.URL.Query()` 获取。

示例代码：

```go
func Hello(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "hello, %s!\n", r.URL.Query().Get(":name"))
}

func main() {
  m := pat.New()
  m.Get("/hello/:name", http.HandlerFunc(Hello))
}
```

缺点是 `r.URL.Query()` 每次都是从原始 querystring 中解析参数，这是对性能不友好的行为，如果包含经过多个中间件，这对性能的影响将更大。速度方面，pat 相较于 httprouter 要慢十倍。

优势: 与 `http.Handle` 兼容.

劣势: 有点慢.

## 如何选择？

如果是传统 Web 应用，服务端进行页面渲染，因为需要复杂的路由，gorilla/mux 是最好的选择。如果是 REST API，httprouter 更加适用，因而，我们将基于 httprouter 完善我们的程序。

## 集成 httprouter

由于 httprouter 与 http.Handler 不兼容，要进行一些调整。实现方案，将中间件栈（http.Handler）包裹，从而实现 httprouter.Handler 接口。

代码如下：

```go
func wrapHandler(h http.Handler) httprouter.Handle {
  return func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    context.Set(r, "params", ps)
    h.ServeHTTP(w, r)
  }
}

func main() {
  db := sql.Open("postgres", "...")
  appC := appContext{db}
  commonHandlers := alice.New(context.ClearHandler, loggingHandler, recoverHandler)
  router := httprouter.New()
  router.GET("/admin", wrapHandler(commonHandlers.Append(appC.authHandler).ThenFunc(appC.adminHandler)))
  router.GET("/about", wrapHandler(commonHandlers.ThenFunc(aboutHandler)))
  router.GET("/", wrapHandler(commonHandlers.ThenFunc(indexHandler)))
  http.ListenAndServe(":8080", router)
}
```

通过 `wrapHandler` 实现将中间件 `http.Handler` 和 `httprouter.Hande` 间的转化，从而实现拥有  httprouter 的良好性能的同时，也能与http.Handler的兼容。

接下来，演示如何在 handler 使用 URL 参数。

创建路由：

```go
router.GET("/teas/:id", wrapHandler(commonHandlers.ThenFunc(appC.teaHandler)))
```

如下代码，创建 `teaHandler`，其中将通过 `id` 从数据库中查询数据。

```go
func (c *appContext) teaHandler(w http.ResponseWriter, r *http.Request) {
  params := context.Get(r, "params").(httprouter.Params)
  tea := getTea(c.db, params.ByName("id"))
  json.NewEncoder(w).Encode(tea)
}
```

## 总结

Go 中的不同 router 的性能差异很大，功能也有差异。最快的路由器并不一定适合你的项目。httprouter 非常适合于 REST API 这样的简单路由，gorilla/mux 更适合具传统的 web 应用。

对于不兼容与 `http.Handler` 的路由实现，可通过类似 `wrapHandler` 实现兼容。

最后，不同 router 方案存储 URL 参数的方式不同，常见的两种方式： `r.URL.Query()` 和 context。在实际使用时，要注意规范一致。

我的博文：[从头构建 Go Web 框架（四）：第三方路由集成](https://www.poloxue.com/posts/2023-09-30-build-your-own-webframework-in-golang-part-4/)
，原文地址: [Part 4: Guide to 3rd Party Routers in Go](https://www.poloxue.com/posts/2023-09-30-build-your-own-webframework-in-golang-part-4/)
