---
title: "从头构建 Go Web 框架（三）：中间件的数据共享"
date: 2023-09-29T13:35:05+08:00
draft: false
comment: true
tags: ["Golang", "Web"]
---

> 本系列文章写于 2014 年，相较于 golang 极短的发展历程，这已经是古董级别的一篇文章了，但 web 框架思想概念依然有效。系统通过这个系列文章，能让大家都现有 Go Web 框架有更深的认识。

本文是 "构建属于自己的 Web 框架" 系列文章中的第二篇，将介绍中间件的最佳实践。

- [第 1 部分：简介](https://www.poloxue.com/posts/2021-10-23-build-your-own-webframework-in-golang)，[Build Your Own Web Framework In Go](https://www.nicolasmerouze.com/build-web-framework-golang)
- [第 2 部分：Go 中间件：最佳实践和示例](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2)，[Part 2: Middlewares in Go: Best practices and examples](https://nicolasmerouze.notion.site/Part-2-Middlewares-in-Go-Best-practices-and-examples-32f41ae0e21b435c86cf9dd38bf0ff65)
- [第 3 部分：中间件数据共享](https://www.poloxue.com/posts/2023-09-30-build-your-own-webframework-in-golang-part-3)，[Part 3: Share Values Between Middlewares](https://nicolasmerouze.notion.site/Part-3-Share-Values-Between-Middlewares-dca6f9448a0c4be68b0da30137c38875)
- [第 4 部分：第三方路由]()，[Part 4: Guide to 3rd Party Routers in Golang](https://nicolasmerouze.notion.site/Part-4-Guide-to-3rd-Party-Routers-in-Go-8ddcca5c360b4539a601ae383c9d7e5d)
- [第 5 部分：使用 MongoDB 实现 JSON-API]()，[How to implement JSON-API standard in MongoDB and Go](https://nicolasmerouze.notion.site/Part-5-How-to-implement-JSON-API-standard-in-MongoDB-and-Go-a3daeead140846e4a6ae1e3c01b47f52)
- [附加福利：上传文件到 s3]()，[Bonus: File Upload REST API with Go and Amazon S3](https://nicolasmerouze.notion.site/Bonus-File-Upload-REST-API-with-Go-and-Amazon-S3-1130fbacad7442c5b0a7df9320d792b4)

我们在 [上文](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2) 中介绍了 middleware 的实现，通过创建 `func (http.Handler) http.Handler` 类型函数，实现了路由或应用间共享代码。案例: 日志（logging） 和 异常恢复（panic recovery）。但是，这两个案例的功能还是相对单一，多数中间件要比它们复杂，需要在中间件间共享数据。

本文将以用户身份验证为例，介绍如何在 middleware 间共享数据。

## 前言

对于面向用户的 web 服务而言，用户认证是几乎每个 `request handler` 都需要的能力，而 middleware 正是为此而生。

如果没有 middleware，则需要在每个 `handler` 中查询数据库得到用户信息，如用户 ID、名称等，认证用户是否合法等。而利用 middleware，则能统一处理。但问题是，下游 handler 还能得到登录用户信息。

如果能解决了 middleware 与 handler 间的数据共享问题，就能实现我们的目标。

## 常见思路

Ruby 中的 Rack 可通过 hash map 类型 `env` 变量存储一个请求周期内的信息。我们也可以用它保存信息。

Javascript 的 `Connect.js` 中间件中有 2 个参数，分别是 `request` 和 `response`。`request` 是个 object，在 javascript 也可以是 map，我们可以用它来存储请求范围内的中间信息。

静态语言也没有什么不同，如 Java 的 Netty 也是通过 map 实现。`handler` 通过访问 context 中的一个 map 属性实现数据共享。

Go 中的一些框架和包是如何做的呢？

## Gorilla

和上面的类似，Gorilla 的 `gorilla/context` 中定义了一个变量 `map[interface{}]interface{}`，可用于存储数据，同时通过 `mutex` 保证线程安全。

定义代码，如下所示：

```go
var (
  mutex sync.RWMutex
  data  = make(map[*http.Request]map[interface{}]interface{})
)
```

如何存储与获取：

```golang
func myHandler(w http.ResponseWriter, r *http.Request) {
  context.Set(r, "foo", "bar")
}

func myOtherHandler(w http.ResponseWriter, r *http.Request) {
  val := context.Get(r, "foo").(string)
}
```

因为 context 中 `data` 中不会被自动清理。如果每个请求都向其新增数据，它将会无限增加。因而，在请求结束后要对 map 进行清理。

**优点：**

- handler 和 middleware 保持不变，调用函数存储和读取即可。

**缺点：**

- 基准测试显示它是性能最差的方案，但实际应用中，性能通常不是核心点；
- 使用 mutex，而非 channel， 但 [Go 文档并不反对 mutex](https://code.google.com/p/go-wiki/wiki/MutexOrChannel)。
- 对于 `interface{}` 类型，需要经过类型断言；

## Goji

Goji 使用一个含有 map[string]interface{} 名为 `Env` 的结构体，无互斥锁。

```golang
func myMiddleware(c *web.C, h http.Handler) http.Handler {
  fn := func (w http.ResponseWriter, r *http.Request) {
    c.Env["name"] = "world"
    h.ServeHTTP(w, r)
  }

  return http.HandlerFunc(fn)
}

func hello(c web.C, w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, %s!", c.Env["name"].(string))
}
```

优点：

- 无互斥锁；

缺点：

- 无法与我们现在的中间件兼容；
- 需要类型断言；

## gocraft/web

[gocraft/web](https://github.com/gocraft/web) 中可以用任何类型作为 `context`  共享数据。没有 map，无类型断言。对于性能来说非常友好，但它是最不兼容的方案。每个应用的 context 都不同的，这将导致你的中间件无法被不同系统复用。

优点：

- 无互斥锁；
- 无类型断言；

缺点：

- 无法与我们的中间件兼容；
- 为此框架而写的代码难以复用；

## go.net/context

Google 内部 context 包。阅读 [context](https://blog.golang.org/context)。一个非常好的解决方案，他们想统一 context 的实现。问题是它是否能与我们的系统很好地配合呢？

## 汇总对比

不同方案对比，如下所示：


| package | mutex | map | struct
| --- | --- | --- | ---
| go.net/context | n | y* | n
| goji | n | y | n
| gin | n | y | n
| martini | n | y | n
| gorilla | y | y | n
| tigertonic | y | n | y
| gocraft/web | n | n | y

> go.net/context 未使用 map，但方法类似。

context 的实现思路都是类似的，但也各有优劣。

为提高性能，一些方案在尽量避免 type assertion、map 或 mutex，但其实性能差异不到 10%。虽然，struct 是最快的，但如果一个请求要 10 ms 的处理时间，这种影响将不到 1%。

毫无疑问，最灵活的实现方案是 map。使用 map 的 context 是 gorilla/context 和 Goji。 

> gorilla/context 是某位 [Go 创建者的方案](http://groups.google.com/group/golang-nuts/msg/e2d679d303aa5d53)，也是最容易实现的。

本文将基于 gorilla/context 实现。

> 如果想将自己的成果分享他人，则要尽量采用标准接口。如 [nosurf](https://github.com/justinas/nosurf) 的 context，与 gorilla/context 是相同概念。在几乎所有框架的项目中使用，而不会出现问题。
>
> 这也是为什么 gorilla/context 系统更优秀，它更易重用中间件。

## context 集成

使用 gorilla/context，我们几乎无需对代码进行任何修改。

在介绍 gorilla/context 时，我说过 context 中的 map 不会自动清理，因此，对于每个 request，map 中都会有新的内容且无限增加。该 package 提供了一个 ClearHandler 解决这问题。

```go
func main() {
  commonHandlers := alice.New(context.ClearHandler, loggingHandler, recoverHandler)
  http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
  http.Handle("/", commonHandlers.ThenFunc(indexHandler))
  http.ListenAndServe(":8080", nil)
}
```

让我们创建一个身份认证 middleware，authhandler，用于用户验证并存储用户信息。

从 header 中获取 token，基于 token 从 database 查找用户。如果认证失败，将返回错误，而不是继续执行下一个 middleware。反之，它将把所获得的用户信息存储在 context，并调用下一个 middleware。

```go
func authHandler(next http.Handler) http.Handler {
  fn := func(w http.ResponseWriter, r *http.Request) {
    authToken := r.Header().Get("Authorization")
    user, err := getUser(authToken)

    if err != nil {
      http.Error(w, http.StatusText(401), 401)
      return
    }

    context.Set(r, "user", user)
    next.ServeHTTP(w, r)
  }

  return http.HandlerFunc(fn)
}

func adminHandler(w http.ResponseWriter, r *http.Requests) {
  user := context.Get(r, "user")
  json.NewEncoder(w).Encode(user)
}

func main() {
  commonHandlers := alice.New(context.ClearHandler, loggingHandler, recoverHandler)
  http.Handle("/admin", commonHandlers.Append(authHandler).ThenFunc(adminHandler))
  http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
  http.Handle("/", commonHandlers.ThenFunc(indexHandler))
  http.ListenAndServe(":8080", nil)
}
```

> 为了简化演示，假设 user 类型是 `map[string]interface{}`。

在 `AdminHandler` 中，我们获取 context 中的用户信息，并写入 `Response` 中。

## 应用级别变量

这种方式的问题是， `getUser` 要访问数据库，而 context 是 request 级别，在每个 request 的 context 中存储数据库连接不是一种好的实现方式。最好的方式是，所有请求共享一个 DB 连接。

如下所示，使用 global/package 变量：

```go
var dbConn *sql.DB

func main() {
  dbConn := sql.Open("postgres", "...")
}
```

应用程序中的任何地方都可访问 `dbConn`，且 `*sql.DB` 是一个并发安全的连接池。以往经验，global 变量不利于维护，存在容易修改，无法追踪的问题。即使有完全的测试，也存在难以重构的问题。

另一个方案是，创建一个 `struct` 变量，管理类似 dbConn 的变量，同时将 handler 和 middle 作为它的方法。为解决上述问题，它将包含 `db` 字段，方法有 `authHandler` 和 `adminHandler`。

稍作变动，即将 `db` 用于 `getUser` 函数，如下所示：

```go
type appContext struct {
  db *sql.DB
}

func (c *appContext) authHandler(next http.Handler) http.Handler {
  fn := func(w http.ResponseWriter, r *http.Request) {
    authToken := r.Header.Get("Authorization")
    user, err := getUser(c.db, authToken)

    if err != nil {
      http.Error(w, http.StatusText(401), 401)
      return
    }

    context.Set(r, "user", user)
    next.ServeHTTP(w, r)
  }

  return http.HandlerFunc(fn)
}

func (c *appContext) adminHandler(w http.ResponseWriter, r *http.Request) {
  user := context.Get(r, "user")
  // Maybe other operations on the database
  json.NewEncoder(w).Encode(user)
}

func main() {
  db := sql.Open("postgres", "...")
  appC := appContext{db}
  commonHandlers := alice.New(context.ClearHandler, loggingHandler, recoverHandler)
  http.Handle("/admin", commonHandlers.Append(appC.authHandler).ThenFunc(appC.adminHandler))
  http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
  http.Handle("/", commonHandlers.ThenFunc(indexHandler))
  http.ListenAndServe(":8080", nil)
}
```

它与中间件系统很好地契合，代码变动很小。类似 gocraft/web 的视线，使用一个 `struct` 管理应用级别变量，但仅适用于应用程序范围。

> 可以将 `getUser` 挂到 `appContext`，使代码更加简洁。或将 *sql.DB 包含在其他自定义结构中，将 `getUser` 添加为这个自定义结构的方法，使用 c.db.getUser(token) 即能完成调用。

## 最后

本文通过 middleware 的数据共享，实现了将用户数据传递到主 handler。实现了以最小的修改将 middleare 和支持 context 的中间件相结合。

下一章主题：[router 路由](https://www.poloxue.com/posts/2023-09-30-build-your-own-webframework-in-golang-part-4/)。

