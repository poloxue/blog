---
title: "基于 net/http 抽象出 go 服务优雅停止的一般思路"
date: 2024-04-14T17:45:43+08:00
draft: false
comment: true
description: "Go 中如何实现优雅停止呢？本文将从 Go 的优雅实现中抽象出一般思路。"
---

和其他语言相比，Go 中有相同也有不同，相同的是实现思路上和其他语言没啥差异，不同在于 Go 采用的是 goroutine + channel 的并发模型，与传统的进程线程相比，实现细节上存在差异。

本文将从实际场景和它的一般实现方式展开，逐步讨论这个话题。

## 简介

什么是优雅停止？在谈优雅停止前，我们可以说说什么是优雅重启，或者说热重启。

简言之，优雅重启就是在服务升级、配置更新时，要重新启动服务，优雅重启就是在服务不中断或连接不丢失的情况下，重启服务。优雅重启的整个流程中，新的进程将在旧的进程停止前启动，旧进程会完成活动中的请求后优雅地关闭进程。

优雅重启是服务开发中一个非常重要的概念，它让我们在不中断服务的情况下，更新代码和修复问题。它在维持高可用性的生产环境中尤其关键。

从上面的这段可知，优雅重启是由两个部分组成，分别是优雅停止和启动。

本文重点介绍优雅停止，而优雅启动的整个流程要借助于外部工具控制，如 k8s 的容器编排。

## 优雅停止

优雅停止，即要在停止服务的同时，保证业务的完整性。从目标上看，优雅停止经历三个步骤：通知服务停止、服务启动清理，等待清理确认退出。

要停止一个服务，首先是通过一些机制告知服务要执行退出前的工作，最常见的就是基于操作系统信号，我们惯例监听的信号主要是两个，分别是由 `kill PID` 发出的 SIGTERM 和 CTRL+C 发出的 SIGINT。 其他信号还有，CTRL+/ 发出的 SIGQUIT。

当接收到指定信号，服务就要停止接受新的请求，且等待当前活动中的请求全部完成后再完全停止服务。

接下来，开始具体的代码实现部分吧。

## 从 HTTP 服务开始

谈优雅重启，最常被引用的案例就是 HTTP 服务，我将通过代码逐步演示这个过程。如下是一个常规 HTTP 服务：

```go
func hello(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello World\n")
}

func main() {
  http.HandleFunc("/", hello)
  log.Println("Starting server on :8080")
  if err := http.ListenAndServe(":8080", nil); err != nil {
      log.Fatal("ListenAndServe: ", err)
  }
}
```

我们通过 `time.Sleep` 增加 hello 的耗时，以便于调试。

```go
func hello(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello World\n")
  time.Sleep(10 * time.Second)
}
```

运行：

```bash
$ go run main.go
```

通过 curl 请求访问 `http://localhost:8080/` ，它进入到 10 秒的处理阶段。假设这时，我们 CTRL+C 请求退出，HTTP 服务会直接退出，我们的 curl 请求被直接中断。

我们可以使用 Go 标准库提供的 `http.Server` 有一个 `Shutdown` 方法，可以安全地关闭服务器而不中断任何活动的连接。而我们要做的，只需在收到停止信号后，执行 `Shutdown` 即可。

信号方面，我们通过 Go 标准库 `signal` 实现，它提供了一个 `Notify` 函数，可与 chan nnel 配合传递信号消息。我们监听的目标信号是 `SIGINT` 和 `SIGTERM`。

重新修改 HTTP 服务入口，使用 `http.Server` 的 `Shutdown` 函数关闭 `Server`。

```go
func main() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", hello)

  server := http.Server{Addr: ":8080", Handler: mux}
  go server.ListenAndServe()
  
  quit := make(chan os.Signal, 1)
  // 注册接收信号的 channel
  signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM) 
  
  <-quit // 等待停止信号
  
  if err := server.Shutdown(context.Background()); err != nil {
    log.Fatal("Shutdown: ", err)
  }
}
```

我们将 `server.ListenAndServe` 运行于另一个 goroutine 中同时忽略了它的返回错误。

通过 `signal.Notify` 注册信号。当收到如 CTRL+C 或 kill PID 发出的中断信号，执行 `serve.Shutdown`，它会通知到 server 停止接收新的请求，并等待活动中的连接处理完成。

现在运行 `go run main.go` 启动服务，执行 `curl` 命令测试接口，在请求还没有返回之时，我们可以通过 CTRL+C 停止服务，它会有一段时间等待，我们可以在这个过程中尝试 `curl` 请求，看它是否还接收新的请求。

如果希望防止程序假死，或者其他问题导致服务长时间无法退出，可通过 `context.WithTimeout` 方法包装下传递给 `Shutdown` 方法的 `ctx` 变量。 

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

if err := server.Shutdown(ctx); err != nil {
  log.Fatal("Shutdown: ", err)
}
```

到这里，我们就介绍完了 Go 标准库 `net/http` 的优雅停止的使用方案。

## 抽象出一个常规方案

如果开发一个非 HTTP 的服务，如何让它支持优雅停止呢？毕竟不是所有项目都是 HTTP 服务，不是所有项目都有现成的框架。

本文开头提到的的三步骤，`net/http` 包的 `Shutdown` 把最核心的服务停止前的清理和等待都已经在内部实现了。我们可解读下它的实现。

进入到 `Shutdown` 的源码中，重点是开头的第一句代码，如下所示：

```go
// future calls to methods such as Serve will return ErrServerClosed.
func (srv *Server) Shutdown(ctx context.Context) error {
  srv.inShutdown.Store(true)
  // ...其他清理代码
  // ...等待活动请求完成并将其关闭
}
```

`inShutdown` 是一个标志位，用于标识程序是否已停止。为了解决并发数据竞争，它的底层类型是 `atomic.bool`，。

在 `server.go` 中的 `Server.Serve` 方法中，通过判断 `inShutdown` 决定是否继续接受新的请求。

```go
func (srv *Server) Serve(l net.Listener)  error {
  // ...
  for {
    rw, err := l.Accept()
    if err != nil {
      if srv.shuttingDown() {
        return ErrServerClosed
      }
  // ...
}
```

我们可以从如上的分析中得知，要让 HTTP 服务支持优雅停止要启动两个 goroutine，`Shutdown` 运行与 main goroutine 中，当接收中停止信号，通过 `inShutdown` 标志位通知运行中的 goroutine。

用简化的代码表示这个一般模式。

```go
var inShutdown bool

func Start() {
  for !inShutdown {
    // running
    time.Sleep(10 * time.Second)
  }
}

func Shutdown() {
  inShutdown = true
}

func main() {
  go Start()

  quit = make(chan signal.Signal, 1)
  signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
  <- quit

  Shutdown()
}
```

大概看起来是那么回事，但这里的代码少了一个步骤，即 `Shutdown` 没有等待 `Start` 完成。

标准库 `net/http` 是通过 for 循环不断检查是否有活动中的连接，如果连接没有进行中请求会将其关闭，直到将所有连接关闭，便会退出 `Shutdown`。

核心代码如下：
```go
func (srv *Server) Shutdown(ctx context.Context) {
  // ...之前的代码

  timer := time.NewTimer(nextPollInterval())
  defer timer.Stop()
  for {
    if srv.closeIdleConns() {
      return lnerr
    }
    select {
    case <-ctx.Done():
      return ctx.Err()
    case <-timer.C:
      timer.Reset(nextPollInterval())
    }
  }
}
```


重点就是那句 `closeIdleConns`，它负责检查是否还有执行中的请求。我就不把这部分的源代码贴出来了。而检查频率是通过 timer 控制的。

现在让简化版等待 `Start` 完成后才退出。我们引入一个名为 `isStop` 的标志位以监控停止状态。


```go
var inShutdown bool
var isStop bool

func Start() {
  for !inShutdown {
    // running
    time.Sleep(10 * time.Second)
  }
  isStop = true
}

func Shutdown() {
  inShutdown = true

  timer := time.NewTimer(time.Millisecond)
  defer timer.Stop()
  for {
    if isStop {
      return
    }
    <- timer.C
    timer.Reset(time.Millisecond))
  }
}
```

如上的代码中，`Start` 函数退出时会执行 `isStop = true` 表明已退出，在 `Shutdown` 中，通过定期检查 `isStop` 等待 `Start` 退出完成。

此外，net/http 的 Shutdown 方法支持接收一个 context.Context 参数，允许实现超时控制，从而防止程序假死或强制关闭。

需要特别指出的是，示例中用的 isStop 和 inShutdown 标志位为非原子类型，在正式场景中，为避免数据竞争，要使用原子操作或其他同步机制。

除了用共享内存标志位在不同进程间传递状态，也可以通过 channel 实现，或你看到过类似如下的形式。

```go
var inShutdown bool

func Start(stop) {
  for !inShutdown {
    // running
    time.Sleep(10 * time.Second)
  }
  stop <- struct{}
}

func Shutdown() {
  inShutdown = true
}

func main() {
  stop := make(chan struct{})
  defer close(stop)

  go Start(stop)

  go func() {
    quit = make(chan signal.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <- quit
    Shutdown()
  }

  <- stop
}
```

`Start` 通过 channel 通知主 goroutine，当触发停止信号，`isShutdown` 通知 `Start` 要停止退出，它成功退出后，通过 `stop <- struct{}` 通知主函数，结束等待。 

## 一点思考

Go 语言支持多种方式在 Goroutine 间传递信息，这催生了多样的优雅停止方案。如果是在涉及多个嵌套 Goroutine 的场景中，我们可以引入 context 来实现多层级的状态和信息传递，确保操作的连贯性和安全性。

然尽管实现方式众多，但其核心思路是一致的，而底层目标始终是我们要保证处理逻辑的完整性。

另外，通过将优雅停止与容器编排技术结合，并为服务添加健康检查，我们能够确保总有服务处于可用状态，实现真正意义上的优雅重启。这不仅提高了服务的可靠性，也优化了资源的利用效率。

## 总结

本文探索了 Go 语言中优雅重启的实现方法，展示了如何通过 http.Server 的 Shutdown 方法安全地重启服务，以及使用 context 控制超时。基于此，我们抽象出了一般服务优雅停止的核心思路。

最后，希望本文对你有所帮助，感谢关注我的公众号。
---

Go 中如何实现优雅停止呢？

和其他语言相比，Go 中有相同也有不同，相同的是实现思路上和其他语言没啥差异，不同在于 Go 采用的是 goroutine + channel 的并发模型，与传统的进程线程相比，实现细节上存在差异。

本文将从实际场景和它的一般实现方式展开，逐步讨论这个话题。

## 简介

什么是优雅停止？在谈优雅停止前，我们可以说说什么是优雅重启，或者说热重启。

简言之，优雅重启就是在服务升级、配置更新时，要重新启动服务，优雅重启就是在服务不中断或连接不丢失的情况下，重启服务。优雅重启的整个流程中，新的进程将在旧的进程停止前启动，旧进程会完成活动中的请求后优雅地关闭进程。

优雅重启是服务开发中一个非常重要的概念，它让我们在不中断服务的情况下，更新代码和修复问题。它在维持高可用性的生产环境中尤其关键。

从上面的这段可知，优雅重启是由两个部分组成，分别是优雅停止和启动。

本文重点介绍优雅停止，而优雅启动的整个流程要借助于外部工具控制，如 k8s 的容器编排。

## 优雅停止

优雅停止，即要在停止服务的同时，保证业务的完整性。从目标上看，优雅停止经历三个步骤：通知服务停止、服务启动清理，等待清理确认退出。

要停止一个服务，首先是通过一些机制告知服务要执行退出前的工作，最常见的就是基于操作系统信号，我们惯例监听的信号主要是两个，分别是由 `kill PID` 发出的 SIGTERM 和 CTRL+C 发出的 SIGINT。 其他信号还有，CTRL+/ 发出的 SIGQUIT。

当接收到指定信号，服务就要停止接受新的请求，且等待当前活动中的请求全部完成后再完全停止服务。

接下来，开始具体的代码实现部分吧。

## 从 HTTP 服务开始

谈优雅重启，最常被引用的案例就是 HTTP 服务，我将通过代码逐步演示这个过程。如下是一个常规 HTTP 服务：

```go
func hello(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello World\n")
}

func main() {
  http.HandleFunc("/", hello)
  log.Println("Starting server on :8080")
  if err := http.ListenAndServe(":8080", nil); err != nil {
      log.Fatal("ListenAndServe: ", err)
  }
}
```

我们通过 `time.Sleep` 增加 hello 的耗时，以便于调试。

```go
func hello(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello World\n")
  time.Sleep(10 * time.Second)
}
```

运行：

```bash
$ go run main.go
```

通过 curl 请求访问 `http://localhost:8080/` ，它进入到 10 秒的处理阶段。假设这时，我们 CTRL+C 请求退出，HTTP 服务会直接退出，我们的 curl 请求被直接中断。

我们可以使用 Go 标准库提供的 `http.Server` 有一个 `Shutdown` 方法，可以安全地关闭服务器而不中断任何活动的连接。而我们要做的，只需在收到停止信号后，执行 `Shutdown` 即可。

信号方面，我们通过 Go 标准库 `signal` 实现，它提供了一个 `Notify` 函数，可与 chan nnel 配合传递信号消息。我们监听的目标信号是 `SIGINT` 和 `SIGTERM`。

重新修改 HTTP 服务入口，使用 `http.Server` 的 `Shutdown` 函数关闭 `Server`。

```go
func main() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", hello)

  server := http.Server{Addr: ":8080", Handler: mux}
  go server.ListenAndServe()
  
  quit := make(chan os.Signal, 1)
  // 注册接收信号的 channel
  signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM) 
  
  <-quit // 等待停止信号
  
  if err := server.Shutdown(context.Background()); err != nil {
    log.Fatal("Shutdown: ", err)
  }
}
```

我们将 `server.ListenAndServe` 运行于另一个 goroutine 中同时忽略了它的返回错误。

通过 `signal.Notify` 注册信号。当收到如 CTRL+C 或 kill PID 发出的中断信号，执行 `serve.Shutdown`，它会通知到 server 停止接收新的请求，并等待活动中的连接处理完成。

现在运行 `go run main.go` 启动服务，执行 `curl` 命令测试接口，在请求还没有返回之时，我们可以通过 CTRL+C 停止服务，它会有一段时间等待，我们可以在这个过程中尝试 `curl` 请求，看它是否还接收新的请求。

如果希望防止程序假死，或者其他问题导致服务长时间无法退出，可通过 `context.WithTimeout` 方法包装下传递给 `Shutdown` 方法的 `ctx` 变量。 

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

if err := server.Shutdown(ctx); err != nil {
  log.Fatal("Shutdown: ", err)
}
```

到这里，我们就介绍完了 Go 标准库 `net/http` 的优雅停止的使用方案。

## 抽象出一个常规方案

如果开发一个非 HTTP 的服务，如何让它支持优雅停止呢？毕竟不是所有项目都是 HTTP 服务，不是所有项目都有现成的框架。

本文开头提到的的三步骤，`net/http` 包的 `Shutdown` 把最核心的服务停止前的清理和等待都已经在内部实现了。我们可解读下它的实现。

进入到 `Shutdown` 的源码中，重点是开头的第一句代码，如下所示：

```go
// future calls to methods such as Serve will return ErrServerClosed.
func (srv *Server) Shutdown(ctx context.Context) error {
  srv.inShutdown.Store(true)
  // ...其他清理代码
  // ...等待活动请求完成并将其关闭
}
```

`inShutdown` 是一个标志位，用于标识程序是否已停止。为了解决并发数据竞争，它的底层类型是 `atomic.bool`，。

在 `server.go` 中的 `Server.Serve` 方法中，通过判断 `inShutdown` 决定是否继续接受新的请求。

```go
func (srv *Server) Serve(l net.Listener)  error {
  // ...
  for {
    rw, err := l.Accept()
    if err != nil {
      if srv.shuttingDown() {
        return ErrServerClosed
      }
  // ...
}
```

我们可以从如上的分析中得知，要让 HTTP 服务支持优雅停止要启动两个 goroutine，`Shutdown` 运行与 main goroutine 中，当接收中停止信号，通过 `inShutdown` 标志位通知运行中的 goroutine。

用简化的代码表示这个一般模式。

```go
var inShutdown bool

func Start() {
  for !inShutdown {
    // running
    time.Sleep(10 * time.Second)
  }
}

func Shutdown() {
  inShutdown = true
}

func main() {
  go Start()

  quit = make(chan signal.Signal, 1)
  signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
  <- quit

  Shutdown()
}
```

大概看起来是那么回事，但这里的代码少了一个步骤，即 `Shutdown` 没有等待 `Start` 完成。

标准库 `net/http` 是通过 for 循环不断检查是否有活动中的连接，如果连接没有进行中请求会将其关闭，直到将所有连接关闭，便会退出 `Shutdown`。

核心代码如下：
```go
func (srv *Server) Shutdown(ctx context.Context) {
  // ...之前的代码

  timer := time.NewTimer(nextPollInterval())
  defer timer.Stop()
  for {
    if srv.closeIdleConns() {
      return lnerr
    }
    select {
    case <-ctx.Done():
      return ctx.Err()
    case <-timer.C:
      timer.Reset(nextPollInterval())
    }
  }
}
```


重点就是那句 `closeIdleConns`，它负责检查是否还有执行中的请求。我就不把这部分的源代码贴出来了。而检查频率是通过 timer 控制的。

现在让简化版等待 `Start` 完成后才退出。我们引入一个名为 `isStop` 的标志位以监控停止状态。


```go
var inShutdown bool
var isStop bool

func Start() {
  for !inShutdown {
    // running
    time.Sleep(10 * time.Second)
  }
  isStop = true
}

func Shutdown() {
  inShutdown = true

  timer := time.NewTimer(time.Millisecond)
  defer timer.Stop()
  for {
    if isStop {
      return
    }
    <- timer.C
    timer.Reset(time.Millisecond))
  }
}
```

如上的代码中，`Start` 函数退出时会执行 `isStop = true` 表明已退出，在 `Shutdown` 中，通过定期检查 `isStop` 等待 `Start` 退出完成。

此外，net/http 的 Shutdown 方法支持接收一个 context.Context 参数，允许实现超时控制，从而防止程序假死或强制关闭。

需要特别指出的是，示例中用的 isStop 和 inShutdown 标志位为非原子类型，在正式场景中，为避免数据竞争，要使用原子操作或其他同步机制。

除了用共享内存标志位在不同进程间传递状态，也可以通过 channel 实现，或你看到过类似如下的形式。

```go
var inShutdown bool

func Start(stop) {
  for !inShutdown {
    // running
    time.Sleep(10 * time.Second)
  }
  stop <- struct{}
}

func Shutdown() {
  inShutdown = true
}

func main() {
  stop := make(chan struct{})
  defer close(stop)

  go Start(stop)

  go func() {
    quit = make(chan signal.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <- quit
    Shutdown()
  }

  <- stop
}
```

`Start` 通过 channel 通知主 goroutine，当触发停止信号，`isShutdown` 通知 `Start` 要停止退出，它成功退出后，通过 `stop <- struct{}` 通知主函数，结束等待。 

## 一点思考

Go 语言支持多种方式在 Goroutine 间传递信息，这催生了多样的优雅停止方案。如果是在涉及多个嵌套 Goroutine 的场景中，我们可以引入 context 来实现多层级的状态和信息传递，确保操作的连贯性和安全性。

然尽管实现方式众多，但其核心思路是一致的，而底层目标始终是我们要保证处理逻辑的完整性。

另外，通过将优雅停止与容器编排技术结合，并为服务添加健康检查，我们能够确保总有服务处于可用状态，实现真正意义上的优雅重启。这不仅提高了服务的可靠性，也优化了资源的利用效率。

## 总结

本文探索了 Go 语言中优雅重启的实现方法，展示了如何通过 http.Server 的 Shutdown 方法安全地重启服务，以及使用 context 控制超时。基于此，我们抽象出了一般服务优雅停止的核心思路。

最后，希望本文对你有所帮助，感谢关注我的公众号。

