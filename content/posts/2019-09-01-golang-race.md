---
title: "Go 的静态检测功能"
date: 2019-09-01T17:32:09+08:00
draft: false
comment: true
tags: ["Golang"]
---

# 译者前言

第三篇 Go 官方博客译文，主要是关于 Go 内置的竞态条件检测工具。它可以有效地帮助我们检测并发程序的正确性。使用非常简单，只需在 go 命令加上 -race 选项即可。

本文最后介绍了两个真实场景下的竞态案例，第一个案例相对比较简单。重点在于第二个案例，这个案例比较难以理解，在原文的基础上，我也简单做了些补充，不知道是否把问题讲的足够清楚。同时，这个案例也告诉我们，任何时候我们都需要重视检测器给我们的提示，因为一不小心，你就可能为自己留下一个大坑。

---

# 概要

在程序世界中，[竞态条件](https://en.wikipedia.org/wiki/Race_condition)是一种潜伏深且很难发现的错误，如果将这样的代码部署线上，常会产生各种谜一般的结果。Go 对并发的支持让我们能非常简单就写出支持并发的代码，但它并不能阻止竞态条件的发生。

本文将会介绍一个工具帮助我们实现它。

Go 1.1 加入了一个新的工具，竞态检测器，它可用于检测 Go 程序中的竞态条件。当前，运行在 x86_64 处理器的 Linux、Mac 或 Windows 下可用。

竞态检测器的实现基于 C/C++ 的 [ThreadSanitizer 运行时库](https://github.com/google/sanitizers)，ThreadSanitier 在 Googgle 已经被用在一些内部基础库以及 [Chromium](http://www.chromium.org/)上，并且帮助发现了很多有问题的代码。

ThreadSanitier 这项技术在 2012 年 9 月被集成到了 Go 上，它帮助检测出了标准库中的 42 个竞态问题。它现在已经是 Go 构建流程中的一部分，当竞态条件出现，将会被它捕获。

# 如何工作

竞态检测器集成在 Go 工具链，当命令行设置了 -race 标志，编译器将会通过代码记录所有的内存访问，何时以及如何被访问，运行时库也会负责监视共享变量的非同步访问。当检测到竞态行为，警告信息会把打印出来。（具体详情阅读 [文章](https://github.com/google/sanitizers/wiki/ThreadSanitizerAlgorithm)）

这样的设计导致竞态检测只能在运行时触发，这也意味着，真实环境下运行 race-enabled 的程序就变得非常重要，但 race-enabled 程序耗费的 CPU 和内存通常是正常程序的十倍，在真实环境下一直启用竞态检测是非常不切合实际的。

是否感受到了一阵凉凉的气息？

这里有几个解决方案可以尝试。比如，我们可以在 race-enabled 的情况下执行测试，负载测试和集成测试是个不错的选择，它偏向于检测代码中可能存在的并发问题。另一种方式，可以利用生产环境的负载均衡，选择一台服务部署启动竞态检测的程序。

# 开始使用

竞态检测器已经集成到 Go 工具链中了，只要设置 -race 标志即可启用。命令行示例如下：

```bash
$ go test -race mypkg
$ go run -race mysrc.go
$ go build -race mycmd
$ go install -race mypkg
```

通过具体案例体验下，安装运行一个命令，步骤如下：

```
$ go get -race golang.org/x/blog/support/racy
$ racy
```

接下来，我们介绍 2 个实际的案例。

# 案例 1：Timer.Reset

这是一个由竞态检测器发现的真实的 bug，这里将演示的是它的一个简化版本。我们通过 timer 实现随机间隔（0-1 秒）的消息打印，timer 会重复执行 5 秒。

首先，通过 time.AfterFunc 创建 timer，定时的间隔从 randomDuration 函数获得，定时函数打印消息，然后通过 timer 的 Reset 方法重置定时器，重复利用。

```go
func main() {
	start := time.Now()
	var t *time.Timer
	t = time.AfterFunc(randomDuration(), func() {
		fmt.Println(time.Now().Sub(start))
		t.Reset(randomDuration())
	})

	time.Sleep(5 * time.Second)
}

func randomDuration() time.Duration {
	return time.Duration(rand.Int63n(1e9))
}
```

我们的代码看起来一切正常。但在多次运行后，我们会发现在某些特定情况下可能会出现如下错误：

```
anic: runtime error: invalid memory address or nil pointer dereference
[signal 0xb code=0x1 addr=0x8 pc=0x41e38a]

goroutine 4 [running]:
time.stopTimer(0x8, 0x12fe6b35d9472d96)
    src/pkg/runtime/ztime_linux_amd64.c:35 +0x25
time.(*Timer).Reset(0x0, 0x4e5904f, 0x1)
    src/pkg/time/sleep.go:81 +0x42
main.func·001()
    race.go:14 +0xe3
created by time.goFunc
    src/pkg/time/sleep.go:122 +0x48
```

什么原因？启用下竞态检测器测试下吧，你会恍然大悟的。

```
$ go run -race main.go
==================
WARNING: DATA RACE
Read by goroutine 5:
  main.func·001()
     race.go:14 +0x169

Previous write by goroutine 1:
  main.main()
      race.go:15 +0x174

Goroutine 5 (running) created at:
  time.goFunc()
      src/pkg/time/sleep.go:122 +0x56
  timerproc()
     src/pkg/runtime/ztime_linux_amd64.c:181 +0x189
==================
```

结果显示，程序中存在 2 个 goroutine 非同步读写变量 t。如果初始定时时间非常短，就可能出现在主函数还未对 t 赋值，定时函数已经执行，而此时 t 仍然是 nil，无法调用 Reset 方法。

我们只要把变量 t 的读写移到主 goroutine 执行，就可以解决问题了。如下：

```go
func main() {
	start := time.Now()
	reset := make(chan bool)
	var t *time.Timer
	t = time.AfterFunc(randomDuration(), func() {
		fmt.Println(time.Now().Sub(start))
		reset <- true
	})
	for time.Since(start) < 5*time.Second {
		<-reset
		t.Reset(randomDuration())
	}
}
```

main goroutine 完全负责 timer 的初始化和重置，重置信号通过一个 channel 负责传递。

当然，这个问题还有个更简单直接的解决方案，避免重用定时器即可。示例代码如下：

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	start := time.Now()
	var f func()
	f = func() {
		fmt.Println(time.Now().Sub(start))
		time.AfterFunc(time.Duration(rand.Int63n(1e9)), f)
	}
	time.AfterFunc(time.Duration(rand.Int63n(1e9)), f)
	time.Sleep(5 * time.Second)
}
```

代码非常简洁易懂，缺点呢，就是效率相对不高。

# 案例 2：ioutil.Discard

这个案例的问题隐藏更深。

ioutil 包中的 Discard 实现了 io.Writer 接口，不过它会丢弃所有写入它的数据，可类比 /dev/null。可在我们需要读取数据但又不准备保存的场景下使用。它常常会和 io.Copy 结合使用，实现抽空一个 reader，如下：

```go
io.Copy(ioutil.Discard, reader)
```

时间回溯至 2011 年，当时 Go 团队注意以这种方式使用 Discard 效率不高，Copy 函数每次调用都会在内部分配 32 KB 的缓存 buffer，但我们只是要丢弃读取的数据，并不需要分配额外的 buffer。我们认为，这种习惯性的用法不应该这样耗费资源。

解决方案非常简单，如果指定的 Writer 实现了 ReadFrom 方法，io.Copy(writer, reader) 调用内部将会把读取工作委托给 writer.ReadFrom(reader) 执行。

Discard 类型增加 ReadFrom 方法共享一个 buffer。到这里，我们自然会想到，这里理论上会存在竞态条件，但因为写入到 buffer 中的数据会被立刻丢弃，我们就没有太重视。

竞态检测器完成后，这段代码立刻被标记为竞态的，查看 [issues/3970](https://github.com/golang/go/issues/3970)。这促使我们再一次思考，这段代码是否真的存在问题呢，但结论依然是这里的竞态不影响程序运行。为了避免这种 "假的警告"，我们实现了 2 个版本的 black_hole buffer，竞态版本和无竞态版本。而无竞态版只会其在启用竞态检测器的时候启用。

black_hole.go，无竞态版本。

```go
// +build race

package ioutil

// Replaces the normal fast implementation with slower but formally correct one.
func blackHole() []byte {
	return make([]byte, 8192)
}
```

black_hole_race.go，竞态版本。

```go
// +build !race

package ioutil

var blackHoleBuf = make([]byte, 8192)

func blackHole() []byte {
	return blackHoleBuf
}
```

但几个月后，[Brad](https://bradfitz.com/) 遇到了一个迷之 bug。经过几天调试，终于确定了原因所在，这是一个由 ioutil.Discard 导致的竞态问题。

实际代码如下：

```go
var blackHole [4096]byte // shared buffer

func (devNull) ReadFrom(r io.Reader) (n int64, err error) {
	readSize := 0
	for {
		readSize, err = r.Read(blackHole[:])
		n += int64(readSize)
		if err != nil {
			if err == io.EOF {
				return n, nil
			}
			return
		}
	}
}
```

Brad 的程序中有一个 trackDigestReader 类型，它包含了一个 io.Reader 类型字段，和 io.Reader 中信息的 hash 摘要。

```go
type trackDigestReader struct {
	r io.Reader
	h hash.Hash
}

func (t trackDigestReader) Read(p []byte) (n int, err error) {
	n, err = t.r.Read(p)
	t.h.Write(p[:n])
	return
}
```

举个例子，计算某个文件的 SHA-1 HASH。

```go
tdr := trackDigestReader{r: file, h: sha1.New()}
io.Copy(writer, tdr)
fmt.Printf("File hash: %x", tdr.h.Sum(nil))
```

某些情况下，如果没有地方可供数据写入，但我们还是需要计算 hash，就可以用 Discard 了。

```go
io.Copy(ioutil.Discard, tdr)
```

此时的 blackHole buffer 并非仅仅是一个黑洞，它同时也是 io.Reader 和 hash.Hash 之间传递数据的纽带。当多个 goroutine 并发执行文件 hash 时，它们全部共享一个 buffer，Read 和 Write 之间的数据就可能产生相应的冲突。No error 并且 No panic，但是 hash 的结果是错的。就是如此可恶。

```go
func (t trackDigestReader) Read(p []byte) (n int, err error) {
	// the buffer p is blackHole
	n, err = t.r.Read(p)
	// p may be corrupted by another goroutine here,
	// between the Read above and the Write below
	t.h.Write(p[:n])
	return
}
```

最终，通过为每一个 io.Discard 提供唯一的 buffer，我们解决了这个 bug，排除了共享 buffer 的竞态条件。代码如下：

```go
var blackHoleBuf = make(chan []byte, 1)

 func blackHole() []byte {
	select {
	case b := <-blackHoleBuf:
		return b
	default:
	}
	return make([]byte, 8192)
}

func blackHolePut(p []byte) {
	select {
	case blackHoleBuf <- p:
	default:
	}
}
```

iouitl.go 中的 devNull ReadFrom 方法也做了相应修正。

```go
func (devNull) ReadFrom(r io.Reader) (n int64, err error) {
buf := blackHole()
	defer blackHolePut(buf)
	readSize := 0
	for {
		readSize, err = r.Read(buf)

	// other
}
```

通过 defer 将使用完的 buffer 重新发送至 blackHoleBuf，因为 channel 的 size 为 1，只能复用一个 buffer。而且通过 select 语句，我们在没有可用 buffer 的情况下，创建新的 buffer。

# 结论

竞态检测器，一个非常强大的工具，在并发程序的正确性检测方面有着很重要的地位。它不会发出假的提示，认真严肃地对待它的每条警示非常必要。但它并非万能，还是需要以你对并发特性的正确理解为前提，才能真正地发挥出它的价值。

试试吧！开始你的 go test -race。

我的博文：[Go 的静态检测功能](https://www.poloxue.com/posts/2019-09-01-golang-race)，译：[Introducing the Go Race Detector](https://blog.golang.org/race-detector)

