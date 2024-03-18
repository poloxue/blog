---
title: "如何防止你的 Goroutine 泄露(二)"
date: 2019-06-17T13:41:42+08:00
draft: false
comment: true
tags: ["Golang"]
---

上篇[文章](https://zhuanlan.zhihu.com/p/74090074)说到，防止 goroutine 泄露可从两个角度出发，分别是代码层面的预防与运行层面的监控检测。今天，我们来谈第二点。

# 简述

前文已经介绍了一种简单检测 goroutine 是否泄露的方法，即通过 runtime.NumGoroutine 获取当前运行中的 goroutine 数量粗略估计。但 NumGoroutine 是否真的能确定我们代码存在泄露，除此之外，还有没有其他更优的方式吗。

注：为了更好的演示效果，下面将会用常驻的 http 作为示例。

# NumGoroutine

runtime.NumGoroutine 可以获取当前进程中正在运行的 goroutine 数量，观察这个数字可以初步判断出是否存在 goroutine 泄露异常。

一个示例，如下：

```go
package main

import (
	"net/http"
	"runtime"
	"strconv"
)

func write(w http.ResponseWriter, data []byte) {
	_, _ = w.Write(data)
}

func count(w http.ResponseWriter, r *http.Request) {
	write([]byte(strconv.Itoa(runtime.NumGoroutine())))
}

func main() {
	http.HandleFunc("/_count", count)
	http.ListenAndServe(":6080", nil)
}
```

功能很简单，设置 `_count` 路由请求处理函数 count，它负责输出服务当前 goroutine 数量。启动服务后访问 localhost:6080/_count 即可。

但只是一个数值，我们就能确认是否泄露了吗？

首先，如果这个数值很大，是不是就能说明出现了泄露。我的答案是否。理由很简单，高并发情况下的 goroutine 数量肯定很高的，但并非出现了泄露，可能只是当前的服务的承载能力还不够。我们可以在数量基础上引入时间，即如果 goroutine 随着时间增加，数量在不断上升，而基本没有下降，基本可以确定存在泄露。我们可以定时采集不同时刻的数据来分析。

# 演示案例

为了更好的演示效果，我们为服务再增加一个处理函数 query， 并绑定路由 /query 上。假设它负责从多个数据表中查出数据返回给用户。这个例子在后面的演示会一直使用。

代码如下：

```go
func query(w http.ResponseWriter, r *http.Request) {
	c := make(chan byte)

	go func() {
		c <- 0x31
	}()

	go func() {
		c <- 0x32
	}()

	go func() {
		c <- 0x33
	}()

	rs := make([]byte, 0)
	for i := 0; i < 2; i++ {
		rs = append(rs, <-c)
	}

	write(w, rs)
}
```

在 query 中，我们启动了 3 个 goroutine 执行数据库查询，通过 channel 传递返回数据。这里的问题是，query 函数中只从 channel 中接收两次数据就退出了循环，这会导致其中一个 goroutine 因缺少接收者而无法释放。

我们可以多次请求 `localhost:6080/query`，然后通过 `_count` 查看服务当前的 goroutine 数量。手动麻烦，可以用 ab 命令进行做个简单压测。

```bash
$ ab -n 1000 -c 100 localhost:6080/query
```

命令的意思是，总共访问 1000 次，并发访问 100 次。

# pprof

前面的例子比较简单，发现泄露后，我们可以立刻确定存在的问题。但如果比较复杂的项目，我们就很难发现问题代码的出现位置了。

如何解决呢？

我们可以引入一个辅助工具，pprof。它是由 Go 官方提供的可用于收集程序运行时报告的工具，其中包含 CPU、内存等信息。当然，也可以获取运行时 goroutine 堆栈信息，如此一来，我们就可以很容易看出哪里导致了 goroutine 泄露。

## runtime/pprof

我们可以再加入一个名为 goroutineStack 的 handler，用于查看程序中 goroutine 的堆栈信息，，地址为 `_goroutine`。

实现代码如下：

```go
import "runtime/pprof"

func goroutineStack(w http.ResponseWriter, r *http.Request) {
	_ = pprof.Lookup("goroutine").WriteTo(w, 1)
}
```

访问 `_goroutine`，将会得到类似如下的信息：

```
goroutine profile: total 1004
948 @ 0x102e70b 0x102e7b3 0x10068ed 0x10066c5 0x1233b37 0x10595d1
#	0x1233b36	main.query.func2+0x36	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:20

45 @ 0x102e70b 0x102e7b3 0x10068ed 0x10066c5 0x1233ae7 0x10595d1
#	0x1233ae6	main.query.func1+0x36	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:16

7 @ 0x102e70b 0x102e7b3 0x10068ed 0x10066c5 0x1233b87 0x10595d1
#	0x1233b86	main.query.func3+0x36	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:24

1 @ 0x102e70b 0x1029ba9 0x1029256 0x108b7da 0x108b8ed 0x108c216 0x112f80f 0x113b348 0x11f5f6a 0x10595d1
#	0x1029255	internal/poll.runtime_pollWait+0x65		/usr/local/go/src/runtime/netpoll.go:173
#	0x108b7d9	internal/poll.(*pollDesc).wait+0x99		/usr/local/go/src/internal/poll/fd_poll_runtime.go:85
#	0x108b8ec	internal/poll.(*pollDesc).waitRead+0x3c		/usr/local/go/src/internal/poll/fd_poll_runtime.go:90
#	0x108c215	internal/poll.(*FD).Read+0x1d5			/usr/local/go/src/internal/poll/fd_unix.go:169
#	0x112f80e	net.(*netFD).Read+0x4e				/usr/local/go/src/net/fd_unix.go:202
#	0x113b347	net.(*conn).Read+0x67				/usr/local/go/src/net/net.go:177
#	0x11f5f69	net/http.(*connReader).backgroundRead+0x59	/usr/local/go/src/net/http/server.go:676

1 @ 0x102e70b 0x1029ba9 0x1029256 0x108b7da 0x108b8ed 0x108c216 0x112f80f 0x113b348 0x11f63ec 0x10fb596 0x10fbf76 0x10fc174 0x119ebbf 0x119eaeb 0x11f315c 0x11f7672 0x11fb23e 0x10595d1
#	0x1029255	internal/poll.runtime_pollWait+0x65		/usr/local/go/src/runtime/netpoll.go:173
#	0x108b7d9	internal/poll.(*pollDesc).wait+0x99		/usr/local/go/src/internal/poll/fd_poll_runtime.go:85
#	0x108b8ec	internal/poll.(*pollDesc).waitRead+0x3c		/usr/local/go/src/internal/poll/fd_poll_runtime.go:90
#	0x108c215	internal/poll.(*FD).Read+0x1d5			/usr/local/go/src/internal/poll/fd_unix.go:169
#	0x112f80e	net.(*netFD).Read+0x4e				/usr/local/go/src/net/fd_unix.go:202
#	0x113b347	net.(*conn).Read+0x67				/usr/local/go/src/net/net.go:177
#	0x11f63eb	net/http.(*connReader).Read+0xfb		/usr/local/go/src/net/http/server.go:786
#	0x10fb595	bufio.(*Reader).fill+0x105			/usr/local/go/src/bufio/bufio.go:100
#	0x10fbf75	bufio.(*Reader).ReadSlice+0x35			/usr/local/go/src/bufio/bufio.go:341
#	0x10fc173	bufio.(*Reader).ReadLine+0x33			/usr/local/go/src/bufio/bufio.go:370
#	0x119ebbe	net/textproto.(*Reader).readLineSlice+0x6e	/usr/local/go/src/net/textproto/reader.go:55
#	0x119eaea	net/textproto.(*Reader).ReadLine+0x2a		/usr/local/go/src/net/textproto/reader.go:36
#	0x11f315b	net/http.readRequest+0x8b			/usr/local/go/src/net/http/request.go:958
#	0x11f7671	net/http.(*conn).readRequest+0x161		/usr/local/go/src/net/http/server.go:966
#	0x11fb23d	net/http.(*conn).serve+0x49d			/usr/local/go/src/net/http/server.go:1788

1 @ 0x102e70b 0x1029ba9 0x1029256 0x108b7da 0x108b8ed 0x108ce80 0x112fd92 0x1142c5e 0x1141967 0x11ff7df 0x121da4c 0x11fed5f 0x11fea16 0x11ff534 0x1233a91 0x102e317 0x10595d1
#	0x1029255	internal/poll.runtime_pollWait+0x65		/usr/local/go/src/runtime/netpoll.go:173
#	0x108b7d9	internal/poll.(*pollDesc).wait+0x99		/usr/local/go/src/internal/poll/fd_poll_runtime.go:85
#	0x108b8ec	internal/poll.(*pollDesc).waitRead+0x3c		/usr/local/go/src/internal/poll/fd_poll_runtime.go:90
#	0x108ce7f	internal/poll.(*FD).Accept+0x19f		/usr/local/go/src/internal/poll/fd_unix.go:384
#	0x112fd91	net.(*netFD).accept+0x41			/usr/local/go/src/net/fd_unix.go:238
#	0x1142c5d	net.(*TCPListener).accept+0x2d			/usr/local/go/src/net/tcpsock_posix.go:139
#	0x1141966	net.(*TCPListener).AcceptTCP+0x46		/usr/local/go/src/net/tcpsock.go:247
#	0x11ff7de	net/http.tcpKeepAliveListener.Accept+0x2e	/usr/local/go/src/net/http/server.go:3232
#	0x11fed5e	net/http.(*Server).Serve+0x22e			/usr/local/go/src/net/http/server.go:2826
#	0x11fea15	net/http.(*Server).ListenAndServe+0xb5		/usr/local/go/src/net/http/server.go:2764
#	0x11ff533	net/http.ListenAndServe+0x73			/usr/local/go/src/net/http/server.go:3004
#	0x1233a90	main.main+0xb0					/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:40
#	0x102e316	runtime.main+0x206				/usr/local/go/src/runtime/proc.go:201

1 @ 0x122ce28 0x122cc30 0x1229694 0x1233723 0x11fc194 0x11fde37 0x11fe8eb 0x11fb3e6 0x10595d1
#	0x122ce27	runtime/pprof.writeRuntimeProfile+0x97			/usr/local/go/src/runtime/pprof/pprof.go:707
#	0x122cc2f	runtime/pprof.writeGoroutine+0x9f			/usr/local/go/src/runtime/pprof/pprof.go:669
#	0x1229693	runtime/pprof.(*Profile).WriteTo+0x3e3			/usr/local/go/src/runtime/pprof/pprof.go:328
#	0x1233722	study/goroutine/leak/06/leak.GoroutineStack+0x92	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/leak/handlers.go:19
#	0x11fc193	net/http.HandlerFunc.ServeHTTP+0x43			/usr/local/go/src/net/http/server.go:1964
#	0x11fde36	net/http.(*ServeMux).ServeHTTP+0x126			/usr/local/go/src/net/http/server.go:2361
#	0x11fe8ea	net/http.serverHandler.ServeHTTP+0xaa			/usr/local/go/src/net/http/server.go:2741
#	0x11fb3e5	net/http.(*conn).serve+0x645				/usr/local/go/src/net/http/server.go:1847
```

首先是第一行，如下：

```
goroutine profile: total 1004
```

统计信息，和 NumGoroutine 的返回结果相同。当前共有 1004 个 goroutine 在运行。

接下来的部分，主要是具体介绍每个 goroutine 的情况，相同函数的 goroutine 会被合并统计，并按数量从大到小排序。输出前三段就是我们在 query 函数中开启的三个 goroutine。

```
948 @ 0x102e70b 0x102e7b3 0x10068ed 0x10066c5 0x1233b37 0x10595d1
#	0x1233b36	main.query.func2+0x36	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:20

45 @ 0x102e70b 0x102e7b3 0x10068ed 0x10066c5 0x1233ae7 0x10595d1
#	0x1233ae6	main.query.func1+0x36	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:16

7 @ 0x102e70b 0x102e7b3 0x10068ed 0x10066c5 0x1233b87 0x10595d1
#	0x1233b86	main.query.func3+0x36	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:24
```

分别是 main.query.func1、main.query.func2 以及 main.query.func3，对应于它们，当前仍在运行中的 goroutine 数量分别是 45、948、7。看样子泄露的 goroutine 函数分布并非均匀。

几个函数都是匿名的，如果我们需要确定具体位置，可以通过堆栈实现。比如 func1，明确指出了位于的所在文件和代码行数。

## http/net/pprof

前面部分是通过自己编写代码把 goroutine 的分析统计指标加入到了 HTTP 服务中。其实，官方已经实现了这个功能，并且涉及的不仅仅是 goroutine，还有 CPU、内存等。

它的操作很简单，我们只需要在服务启动时导入 net/http/pprof 即可。接着访问地址 /debug/pprof/goroutine?debug=1，将会可以看到与上一节输出的相同内容。

# gops

熟悉 Java 的朋友都知道 jps 这个命令。通过它，我们可以查看当前机器上有哪些 Java 程序在运行。Go 也有类似的命令，[gops]()，它支持列出当前环境下的 Go 进程，并支持对 Go 程序的诊断。默认情况下，gops 可列出并不支持对进程进行成诊断。

今天，我们将只看它和 goroutine 相关的部分。

一个示例，如下：

```bash
$ gops
97778 96800 gops    go1.11.1 /usr/local/go/bin/gops
97605 73594 leaker* go1.11.1 /Users/polo/Public/Work/go/src/study/goroutine/leak/06/leaker
```

我的环境下当前只有两个 go 进程在运行。

仔细观察后，我们会发现 leaker 进程相比 gops 后面多个 * 的标号，而 * 表示这个程序支持通过 gops 诊断。这是因为我们在 leaker 加入了诊断支持的代码，如下：

```go
func main() {
	if err := agent.Listen(agent.Options{ShutdownCleanup: true}); err != nil {
		log.Fatalln(err)
	}

	...
}
```

执行如下命令，查看当前的 goroutine 数量。

```bash
$ gops stats 97605
goroutines: 1004
OS threads: 14
GOMAXPROCS: 8
num CPU: 8
```

其中，97605 是进程 PID。

结果显示，当前在运行的 goroutine 有 1004 个。而且，我们还注意到 OS 级别的线程才 14 个，可见 goroutine 的轻量。

gops 也可以查看堆栈，我们只需执行 gops stack PID 即可，这个就不具体演示了。要说明的是，这种方式并不会对运行相同函数的 goroutine 做聚合统计，不知道是我没找到还是本身不支持。如果的确不支持，也可以自己聚合，但毕竟没那么方便。

# Leak Test

除了出现问题后的检测调试，但如果我们能把泄露检测过程加入到自动化测试中，在正式上线前就避免，岂不是更完美。我们可以通过一个开源包实现，包的名称是 [leaktest](https://github.com/fortytw2/leaktest)，即泄露测试的意思。

利用 leaktest，我们测试下前面写的 http 处理函数 query。因为要检测 handler 是否泄露，如果经过网络就会丢失服务端的相关信息，这时，我们可以借助 Go 中的 net/http/test 包完成测试。

代码如下：

```go
func Test_Query(t *testing.T) {
	defer leaktest.Check(t)()

	//创建一个请求
	req, err := http.NewRequest("GET", "/query", nil)
	if err != nil {
		t.Fatal(err)
	}

	rr := httptest.NewRecorder()

	//直接使用 query(rr,req)
	query(rr, req)

	// 其他测试
	// ...
}
```

测试执行输出如下：

```bash
=== RUN   Test_Query
--- FAIL: Test_Query (5.01s)
    leaktest.go:162: leaktest: context canceled
    leaktest.go:168: leaktest: leaked goroutine: goroutine 20 [chan send]:
        study/goroutine/leak/06.query.func2(0xc0001481e0)
        	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:24 +0x37
        created by study/goroutine/leak/06.query
        	/Users/polo/Public/Work/go/src/study/goroutine/leak/06/main.go:23 +0x7e
FAIL
```

从输出信息中，我们可以明确地知道出现了泄露，并且通过输出堆栈很快就能定位出现问题的代码。测试代码非常简单，在测试函数开始通过 defer 执行 leaktest 的 Check。

它提供的三个检测函数，分别是 Check、CheckTimeout 和 CheckContext，从前到后的实现一个比一个底层。Check 默认会等待五秒再执行检测，如果需要改变这个时间，可以使用 CheckTimeout 函数。

leaktest 的实现原理也和堆栈有关，源码不多，如果有兴趣可以读读，[源码文件地址](https://github.com/fortytw2/leaktest/blob/master/leaktest.go)。

# 总结

本系列文章分别从代码实现和监控检测两个角度介绍了如何避免 goroutine 的泄露。Go 的并发降低了并发程序的开发难度，但并发一直都是个比较复杂的话题，为了用好它，必要学习还是不可缺少的。

博文地址：[如何防止你的 Goroutine 泄露](https://www.poloxue.com/posts/2019-06-17-prevent-goroutine-from-leaking-part-2/)

# 参考资料

[Goroutine leak](https://medium.com/golangspec/goroutine-leak-400063aef468)  
[gops 工作原理](https://blog.wolfogre.com/posts/mechanism-of-gops/)  
[gops - Go 语言程序查看和诊断工具](https://www.cnblogs.com/snowInPluto/p/7785651.html)  
[gops — Go 程序诊断分析工具](https://shockerli.net/post/golang-tool-gops/)  
[性能调式：分析并优化 Go 程序](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/13.10.md)  
[Debugging Go Routine leaks](https://blog.minio.io/debugging-go-routine-leaks-a1220142d32c)  
[视频-Debugging Go routine leaks](https://www.youtube.com/watch?v=hWo0FEVr92A&feature=youtu.be)  
[HTTP 测试辅助工具](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter09/09.6.html)  

