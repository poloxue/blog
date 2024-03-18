---
title: "Go 通过 Context 实现并发控制"
date: 2019-07-12T16:55:58+08:00
draft: false
comment: true
tags: ["Golang"]
---

## 译者前言

第二篇官方博客的翻译，主要是关于 Go 并发控制的 context 包。

总体来说，我认为[上一篇](https://juejin.im/post/5d01177a5188254b9000975c)才是 Go 并发的基础与核心。context 是在前章基础之上，为 goroutine 控制而开发的一套便于使用的库。毕竟，在不同的 goroutine 之间只传递 done channel，包含信息量确实是太少。

文章简单介绍了 context 提供的方法，以及简单介绍它们如何使用。接着，通过一个搜索的例子，介绍了在真实场景下的使用。

文章的尾部部分说明了，除了官方实现的 context，也有一些第三方的实现，比如 [github.com/context](http://github.com/gorilla/context) 和 [Tomb](https://github.com/go-tomb/tomb)，但这些在官方 context 出现之后就已经停止更新了。其实原因很简单，毕竟一般都是官方更强大。之前，go 模块管理也是百花齐放，但最近官方推出自己的解决方案，或许不久，其他方式都将会淘汰。

其实，我觉得这篇文章并不好读，感觉不够循序渐进。突然的一个例子或许会让人有点懵逼。

正文如下：

---

Go 的服务中，每个请求都会有独立的 goroutine 处理，每个 goroutine 通常会启动新的 goroutine 执行一些额外的工作，比如进行数据库或 RPC 服务的访问。同请求内的 goroutine 需能共享请求数据访问，比如，用户认证，授权 token，以及请求截止时间。如果请求取消或发生超时，请求范围内的所有 goroutine 都应立刻退出，进行资源回收.

在 Google，我们开发了一个 context 的包，通过它，我们可以非常方便地在请求内的 goroutine 之间传递请求数据、取消信号和超时信息。详情查看 [context](https://golang.org/pkg/context)。

本文将会具体介绍 context 包的使用，并提供一个完整的使用案例。

## Context

context 的核心是 Context 类型。定义如下：

```go
// A Context carries a deadline，cancellation signal，and request-scoped values
// across API. Its methods are safe for simultaneous use by multiple goroutines
// 一个 Context 可以在 API (无论是否是协程间) 之间传递截止日期、取消信号、请求数据。
// Context 中的方法都是协程安全的。
type Context interface {
    // Done returns a channel that is closed when this context is cancelled
    // or times out.
    // Done 方法返回一个 channel，当 context 取消或超时，Done 将关闭。
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed
    // 在 Done 关闭后，Err 可用于表明 context 被取消的原因
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    // 到期则取消 context
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none
    Value(key interface{}) interface{}
}
```

介绍比较简要，详细信息查看 [godoc](https://golang.org/pkg/context)。

Done 方法返回的是一个 channel，它可用于接收 context 的取消信号。当 channel 关闭，监听 Done 信号的函数会立刻放弃当前正在执行的工作并返回。Err 方法返回一个 error 变量，从它之中可以知道 context 为什么被取消。[pipeline and cancelation](https://blog.golang.org/pipelines) 一文对 Done channel 作了详细介绍。

为什么 Context 没有 cancel 方法，它的原因与 Done channel 只读的原因类似，即接收取消信号的 goroutine 通过不会负责取消信号的发出。特别是，当父级启动子级 goroutine 来执行操作，子级是无法取消父级的。反之，WithCancel 方法（接下来介绍）提供了一种方式取消新创建的 Context。

Context 是协程并发安全的。我们可以将 Context 传递给任意数量的 goroutine，通过 cancel 可以给所有的 goroutine 发送信号。

Deadline 方法可以让函数决定是否需要启动工作，如果剩余时间太短，那么启动工作就不值得了。在代码中，我们可以通过 deadline 为 IO 操作设置超时时间。

Value 方法可以让 context 在 goroutine 之间共享请求范围内的数据，这些数据需要是协程并发安全的。

## 派生 Context

context 包提供了多个函数从已有的 Context 实例派生新的 Context。这些 Context 将会形成一个树状结构，只要一个 Context 取消，派生的 context 将都被取消。

Background 函数返回的 Context 是任何 Context 根，并且不可以被取消。

```
// Background returns an empty Context. It is never canceled, has no deadline,
// and has no values. Background is typically used in main, init, and tests，
// and as the top-level Context for incoming requests.
// Background 函数返回空 Context，并且不可以取消，没有最后期限，没有共享数据。Background 仅仅会被用在 main、init 或 tests 函数中。
func Background() Context
```

WithCancel 和 WithTimeout 会派生出新的 Context 实例，派生实例比父级更早被取消。与请求关联的 Context 实例，在请求处理完成后将被取消。当遇到多副本的数据请求时，WithCancel 可用于取消多余请求。在请求后端服务时，WithTimeout 可用于设置超时时间。

```go
// WithCancel returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed or cancel is called.
// WithCanal 返回父级 Context 副本，当父级的 Done channel 关闭或调用 cancel，它的 Done channel 也会关闭。
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

// A CancelFunc cancels a Context.
// CancelFunc 用于取消 Context
type CancelFunc func()

// WithTimeout returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed, cancel is called, or timeout elapses. The new
// Context's Deadline is the sooner of now+timeout and the parent's deadline, if
// any. If the timer is still running, the cancel function releases its
// resources.
// 返回父级 Context 副本和 CancelFunc，三种情况，它的 Done 会关闭，分别是父级 Done 关闭，cancel 被调用，和达到超时时间。
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

WithValue 提供了一种方式，通过 Context 传递请求相关的数据

```go
// WithValue returns a copy of parent whose Value method returns val for key.
func WithValue(parent Context, key interface{}, val interface{}) Context
```

context 如何使用呢？最好的方式，通过一个案例演示。

## 案例：Google Web 搜索

演示一个案例，实现一个 HTTP 服务，处理类似 /search?q=golang&timeout=1s 的请求。timeout 表示如果请求处理时间超过了指定时间，取消执行。

代码主要涉及 3 个 package，分别是：

- server，主函数入口和 /search 处理函数；
- userip，实现从 request 的 context 导出 user ip 的公共函数；
- google，实现了 Search 函数，负责向 Google 发送搜索请求；

开始介绍!

## server

server 负责处理类似 /search?q=golang 的请求，返回 golang 搜索结果，handleSearch 是实际的处理函数，它首先初始化了一个 Context，命名为 ctx，通过 defer 实现函数退出 cancel。如果请求参数含有 timeout，通过 WithTimeout 创建 context，在超时后，Context 将自动取消。

```go
func handleSearch(w http.ResponseWriter, req *http.Request) {
    // ctx is the Context for this handler. Calling cancel closes the
    // cxt.Done channel, which is the cancellation signal for requests
    // started by this handler
    var (
        ctx context.Context
        cancel context.Context
    )

    timeout, err := time.ParseDuration(req.FromValue("timeout"))
    if err != nil {
        // the request has a timeout, so create a context that is
        // canceled automatically when the timeout expires.
        ctx, cancel = context.WithTimeout(context.Background(), timeout)
    } else {
        ctx, cancel = context.WithCancel(context.Background(), timeout)
    }
    defer cancel() // Cancel ctx as soon as handlSearch returns.
```

下一步，处理函数会从请求中获取查询关键词和客户端 IP，客户端 IP 的获取通过调用 userip 包函数实现。同时，由于后端服务的请求也需要客户端 IP，故而将其附在 ctx 上。

```go
    // Check the search query
    query := req.FormValue("q")
    if query == "" {
        http.Error(w, "no query", http.StatusBadRequest)
        return
    }

    // Store the user IP in ctx for use by code in other packages.
    userIP, err := userip.FormRequest(req)
    if err != nil {
        http.Error(w, e.Error(), http.StatusBadRequest)
        return
    }

    ctx = userip.NewContext(ctx, userIP)
```

调用 google.Search，并传入 ctx 和 query 参数。

```go
    // Run the Google search and print the results
    start := time.Now()
    results, err := google.Search(ctx, query)
    elapsed := time.Since(start)
```

搜索成功后，handler 渲染结果页面。

```go
    if err := resultsTemplate.Execute(w, struct{
        Results     google.Results
        Timeout, Elapsed time.Duration
    }{
        Results: results,
        Timeout: timeout,
        Elaplsed: elaplsed,
    }); err != nil {
        log.Print(err)
        return
    }
```

### Package userip

userip 包中提供了两个函数，负责从请求中导出用户 IP 和将用户 IP 绑定 Context 上。 Context 中包含 key-value 映射，key 与 value 的类型都是 interface{}，key 必须支持相等比较，value 要是协程并发安全的。userip 包通过对 Context 中的 value ，即 client IP 执行了类型转化，隐藏了 map 的细节。为了避免 key 的冲突，userip 定义了一个不可导出的类型 key。

```go
// The key type is unexported to prevent collision with context keys defined in
// other package
type key int

// userIPkey is the context key for the user IP address. Its value of zero is
// arbitrary. If this package defined other context keys, they would have
// different integer values.
const userIPKye key = 0
```

函数 FromRequest 负责从 http.Request 导出用户 IP:

```go
func FromRequest(req *http.Request) (net.IP, error) {
    ip, _, err := net.SplitHostPort(req.RemoteAddr)
    if err != nil {
        return nil, fmt.Errorf("userip: %q is not IP:port", req.RemoteAddr)
    }
```

函数 NewContext 生成一个带有 userIP 的 Context:

```go
func NewContext(ctx context.Context, userIP net.IP) context.Context {
    return context.WithValue(ctx, userIPKey, userIP)
}
```

FromContext 负责从 Context 中导出 userIP:

```go
func FromContext(ctx context.Context) (net.IP. bool) {
    // ctx.Value returns nil if ctx has no value for the key;
    // the net.IP type assertion returns ok=false for nil
    userIP, ok := ctx.Value(userIPKey).(net.IP)
    return userIP, ok
}
```

### Package google

google.Search 负责 Google Web Search 接口的请求，以及接口返回 JSON 数据的解析。它接收 Context 类型参数 ctx，如果 ctx.Done 关闭，即使请求正在运行也将立刻返回。

查询的请求参数包括 query 关键词和用户 IP。

```go
func Search(ctx context.Context, query string) (Results, error) {
    // Prepare the Google Search API request
    req, err := http.NewRequest("GET", "http://ajax.googleapis.com/ajax/services/search/web?v=1.0", nil)
    if err != nil {
        return nil, err
    }
    q := req.URL.Query()
    q.Set("q", query)

    // If ctx is carrying the user IP address, forward it to the server
    // Google APIs use the user IP to distinguish server-initiated requests 
    // from end-users requests
    if userIP, ok := userip.FromContext(ctx); ok {
        q.Set("userip", userIP.String())
    }
    req.URL.RawQuery = q.Encode()
```

Search 函数使用了一个帮助函数，httpDo，负责发起 HTTP 请求，如果 ctx.Done 关闭，即使请求正在执行，也会被关闭。Search 传递了一个闭包函数给 httpDo 处理响应结果。

```go
    var results Results
    err = httpDo(ctx, req, func(resp *http.Response, err error) error {
        if err != nil {
            return err
        }
        defer resp.Body.Close()

        // Parse the JSON search result.
        // https://developers.google.com/web-search/docs/#fonje
        var data struct {
            ResponseData struct {
                Results []struct {
                    TitleNoFormatting string
                    URL               string
                }
            }
        }
        if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
            return err
        }

        for _, res := range data.ResponseData.Results {
            results = append(results, Result{Title: res.TitleNoFormatting, URL: res.URL})
        }

        return nil
    })

    return results, err
```

httpDo 函数开启一个新的 goroutine 负责 HTTP 请求执行和响应结果处理。如果在 goroutine 退出前，即请求还没执行结束，如果 ctx.Done 关闭，请求执行将被取消。

```go
func httpDo(ctx context.Context, req *http.Request, f func(*http.Request, error) error) error {
    // Run the HTTP request in a goroutine and pass the response to f.
    c := make(chan error, 1)
    req := req.WithContext(ctx)
    go func() { c <- f(http.DefaultClient.Do(req)) }()
    select {
    case <-ctx.Done():
        <- c
        return ctx.Err
    case  err := <-c:
        return err
    }
}
```

## 基于 Context 调整代码 

许多服务端框架都提供了相应的包和数据类型进行请求的数据传递。我们可以基于 Context 接口编写新的实现代码，完成框架与处理函数的连接。

```
译者注：下面介绍的就是开发说的两个 context 的第三方实现，其中有些内容需要简单了解下它们才能完全看懂。
```

例如，Gorilla's 的 [context](https://github.com/gorilla/context) 通过在请求上提供 key value 映射实现关联数据绑定。在 [gorilla.go](https://blog.golang.org/context/gorilla/gorilla.go)，提供了 Context 的实现，它的 Value 方法返回的值和一个具体的 HTTP 请求关联。

其他一些包提供与 Context 类似的取消支持。例如，[Tomb](https://godoc.org/gopkg.in/tomb.v2) 中有 Kill 方法通过关闭 Dying channel 实现取消信号发出。Tomb 也提供了方法用于等待 goroutine 退出，与 sync.WaitGroup 类似。在 [tomb.go](https://blog.golang.org/context/tomb/tomb.go) 中，提供了一种实现，当父 Context 取消或 Tomb 被 kill时，当前 Context 将会取消。

## 总结

在 Google，对于接收或发送请求类的函数，我们要求必须要将 Context 作为首个参数进行传递。如此，即使不同团队的 Go 代码也可以工作良好。Context 非常便于 goroutine 的超时与取消控制，以及确保重要数据的安全传递，比如安全凭证。

基于 Context 的服务框架需要实现 Context，帮助连接框架和使用方，使用方期望从框架接收 Context 参数。而客户端库，则与之相反，它从调用方接收 Context 参数。context 通过为请求数据与取消控制建立通用接口，实现包开发者们可以非常轻松地共享自己的代码，以及打造出更具扩展性的服务。

我的博文：[Go 通过 Context 实现并发控制](https://www.poloxue.com/posts/2019-07-12-concurrent-using-context)，译：[blog.golang.org/context](https://blog.golang.org/context)

