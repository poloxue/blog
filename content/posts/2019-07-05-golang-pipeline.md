---
title: "Go 如何构建并发 Pipeline"
date: 2019-07-05T16:52:21+08:00
draft: false
comment: true
tags: ["Golang"]
---


## 译者前言

这篇文章来自 Go 官网，不愧是官方的博客，写的非常详细。在开始翻译这篇文章前，先简单说明两点。

首先，这篇文章我之前已经翻译过一遍，但最近再读，发现之前的翻译真是有点烂。于是，决定在完全不参考之前译文的情况下，把这篇文章重新翻译一遍。

其二，文章中有一些专有名字，计划还是用英文来表达，以保证原汁原味，比如 pipeline（管道）、stage (阶段)、goroutine (协程)、channel (通道)。

关于它们之间的关系，按自己的理解简单画了张草图，希望能帮助更好地理解它们之间的关系。如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-07-05-golang-pipeline-01.jpeg)

强调一点，如果大家在阅读这篇文章时，感到了迷糊，建议可以回头再看一下这张图。

翻译的正文部分如下。

----

Go 的并发原语使我们非常轻松地就构建出可以高效利用 IO 和多核 CPU 的流式数据 pipeline。这篇文章将会此为基础进行介绍。在这个过程中，我们将会遇到一些异常情况，关于它们的处理方法，文中也会详细介绍。

## 什么是管道（pipeline）

关于什么是管道， Go 中并没有给出明确的定义，它只是众多并发编程方式中的一种。非正式的解释，我们理解为，它是由一系列通过 chanel 连接起来的 stage 组成，而每个 stage 都是由一组运行着相同函数的 goroutine 组成。每个 stage 的 goroutine 通常会执行如下的一些工作：

- 从上游的输入 channel 中接收数据；
- 对接收到的数据进行一些处理，（通常）并产生新的数据；
- 将数据通过输出 channel 发送给下游；

除了第一个 stage 和最后一个 stage ，每个 stage 都包含一定数量的输入和输出 channel。第一个 stage 只有输出，通常会把它称为 "生产者"，最后一个 stage 只有输入，通常我们会把它称为 "消费者"。

我们先来看一个很简单例子，通过它来解释上面提到那些与 pipeline 相关的概念和技术。了解了这些后，我们再看其它的更实际的例子。

## 计算平方数

一个涉及三个 stage 的 pipeline。

第一个 stage，gen 函数。它负责将把从参数中拿到的一系列整数发送给指定 channel。它启动了一个 goroutine 来发送数据，当数据全部发送结束，channel 会被关闭。

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}
```

第二个 stage，sq 函数。它负责从输入 channel 中接收数据，并会返回一个新的 channel，即输出 channel，它负责将经过平方处理过的数据传输给下游。当输入 channel 关闭，并且所有数据都已发送到下游，就可以关闭这个输出 channel 了。

```go
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```

main 函数负责创建管道并执行最后一个 stage 的任务。它将从第二个 stage 接收数据，并将它们打印出来，直到 channel 关闭。

```go
func main() {
    // Set up the pipeline.
    c := gen(2, 3)
    out := sq(c)

    // Consume the output.
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}
```

既然，sq 的输入和输出的 channel 类型相同，那么我们就可以把它进行组合，从而形成多个 stage。比如，我们可以把 main 函数重写为如下的形式：

```go
func main() {
    // Set up the pipeline and consume the output.
    for n := range sq(sq(gen(2, 3))) {
        fmt.Println(n) // 16 then 81
    }
}
```

## 扇出和扇入（Fan-out and Fan-in）

当多个函数从一个 channel 中读取数据，直到 channel 关闭，这称为扇出 fan-out。利用它，我们可以实现了一种分布式的工作方式，通过一组 workers 实现并行的 CPU 和 IO。

当一个函数从多个 channel 中读取数据，直到所有 channel 关闭，这称为扇入 fan-in。扇入是通过将多个输入 channel 的数据合并到同一个输出 channel 实现的，当所有的输入 channel 关闭，输出的 channel 也将关闭。

我们来改变一下上面例子中的管道，在它上面运行两个 sq 函数试试。它们将都从同一个输入 channel 中读取数据。我们引入了一个新的函数，merge，负责 fan-in 处理结果，即 merge 两个 sq 的处理结果。

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    // 分布式处理来自 in channel 的数据
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    // 从 channel c1 和 c2 的合并后的 channel 中接收数据
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}
```

merge 函数负责将从一系列输入 channel 中接收的数据合并到一个 channel 中。它为每个输入 channel 都启动了一个 goroutine，并将它们中接收到的值发送到惟一的输出 channel 中。在所有的 goroutines 启动后，还会再另外启动一个 goroutine，它的作用是，当所有的输入 channel 关闭后，负责关闭唯一的输出 channel 。

在已关闭的 channel 发送数据将导致 panic，因此要保证在关闭 channel 前，所有数据都发送完成，是非常重要的。sync.WaitGroup 提供了一种非常简单的方式来完成这样的同步。

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    // 为每个输入 channel 启动一个 goroutine
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    // 启动一个 goroutine 负责在所有的输入 channel 关闭后，关闭这个唯一的输出 channel
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

## 中途停止

管道中的函数包含一个模式:

- 当数据发送完成，每个 stage 都应该关闭它们的输入 channel；
- 只要输入 channel 没有关闭，每个 stage 就要持续从中接收数据；

我们可以通过编写 range loop 来保证所有 goroutine 是在所有数据都已经发送到下游的时候退出。

但在一个真实的场景下，每个 stage 都接收完 channel 中的所有数据，是不可能的。有时，我们的设计是：接收方只需要接收数据的部分子集即可。更常见的，如果 channel 在上游的 stage 出现了错误，那么，当前 stage 就应该提早退出。无论如何，接收方都不该再继续等待接收 channel 中的剩余数据，而且，此时上游应该停止生产数据，毕竟下游已经不需要了。

我们的例子中，即使 stage 没有成功消费完所有的数据，上游 stage 依然会尝试给下游发送数据，这将会导致程序永久阻塞。

```go
    // Consume the first value from the output.
    // 从 output 中接收了第一个数据
    out := merge(c1, c2)
    fmt.Println(<-out) // 4 or 9
    return
    // Since we didn't receive the second value from out,
    // one of the output goroutines is hung attempting to send it.
    // 我们并没有从 out channel 中接收第二个数据，
    // 所以上游的其中一个 goroutine 在尝试向下游发送数据时将会被挂起。
}
```

这是一种资源泄露，goroutine 是需要消耗内存和运行时资源的，goroutine 栈中的堆引用信息也是不会被 gc。

我们需要提供一种措施，即使当下游从上游接收数据时发生异常，上游也能成功退出。一种方式是，把 channel 改为带缓冲的 channel，这样，它就可以承载指定数量的数据，如果 buffer channel 还有空间，数据的发送将会立刻完成。

```go
// 缓冲大小 2 buffer size 2 
c := make(chan int, 2)
// 发送立刻成功 succeeds immediately 
c <- 1
// 发送立刻成功 succeeds immediately
c <- 2 
//blocks until another goroutine does <-c and receives 1
// 阻塞，直到另一个 goroutine 从 c 中接收数据
c <- 3
```

如果我们在创建 channel 时已经知道将发送的数据量，就可以把前面的代码简化一下。比如，重写 gen 函数，将数据都发送至一个 buffer channel，这还能避免创建新的 goroutine。

```go
func gen(nums ...int) <-chan int {
    out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
    return out
}
```

译者按：channel 关闭后，不可再写入数据，否则会 panic，但是仍可读取已发送数据，而且可以一直读取 0 值。

继续往下游 stage，将又会返回到阻塞的 goroutine 中，我们也可以考虑给 merge 的输出 channel 加点缓冲。

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup

    // enough space for the unread inputs
    // 给未读的输入 channel 预留足够的空间
    out := make(chan int, 1)    
    // ... the rest is unchanged ...
```

虽然通过这个方法，我们能解决了 goroutine 阻塞的问题，但是这并非一个优秀的设计。比如 merge 中的 buffer 的大小 1 是基于我们已经知道了接下来接收数据的大小，以及下游将能消费的数量。很明显，这种设计非常脆弱，如果上游多发送了一些数据，或下游并没接收那么多的数据，goroutine 将又会被阻塞。

因而，当下游不再准备接收上游的数据时，需要有一种方式，可以通知到上游。

## 明确的取消

如果 main 函数在没把 out 中所有数据接收完就退出，它必须要通知上游停止继续发送数据。如何做到？我们可以在上下游之间引入一个新的 channel，通常称为 done。

示例中有两个可能阻塞的 goroutine，所以， done 需要发送两个值来通知它们。

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the first value from output.
    done := make(chan struct{}, 2)
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // Tell the remaining senders we're leaving.
    // 通知发送方，我们已经停止接收数据了
    done <- struct{}{}
    done <- struct{}{}
}
```

发送方 merge 用 select 语句替换了之前的发送操作，它负责通过 out channel 发送数据或者从 done 接收数据。done 接收的值是没有实际意义的，只是表示 out 应该停止继续发送数据了，用空 struct 即可。output 函数将会不停循环，因为上游，即 sq ，并没有阻塞。我们过会再讨论如何退出这个循环。

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed or it receives a value
    // from done, then output calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            select {
            case out <- n:
            case <-done:
            }
        }
        wg.Done()
    }
    // ... the rest is unchanged ...
```

这种方法有个问题，下游只有知道了上游可能阻塞的 goroutine 数量，才能向每个 goroutine 都发送了一个 done 信号，从而确保它们都能成功退出。但多维护一个 count 是很令人讨厌的，而且很容易出错。

我们需要一种方式，可以告诉上游的所有 goroutine 停止向下游继续发送信息。在 Go 中，其实可通过关闭 channel 实现，因为在一个已关闭的 channel 接收数据会立刻返回，并且会得到一个零值。

这也就意味着，main 仅需通过关闭 done channel，就可以让所有的发送方解除阻塞。关闭操作相当于一个广播信号。为确保任意返回路径下都成功调用，我们可以通过 defer 语句关闭 done。

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c or done is closed, then calls
    // wg.Done.
    // 为每个输入 channel 启动一个 goroutine，将输入 channel 中的数据拷贝到
    // out channel 中，直到输入 channel，即 c，或 done 关闭。
    // 接着，退出循环并执行 wg.Done()
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            select {
            case out <- n:
            case <-done:
                return
            }
        }
    }
    // ... the rest is unchanged ...
```

同样地，一旦 done 关闭，sq 也将退出。sq 也是通过 defer 语句来确保自己的输出 channel，即 out，一定被成功关闭释放。

```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}
```

都这里，Go 中如何构建一个 pipeline，已经介绍的差不多了。

简单总结下如何正确构建一个 pipeline。

- 当所有的发送已经完成，stage 应该关闭输出 channel；
- stage 应该持续从输入 channel 中接收数据，除非 channel 关闭或主动通知到发送方停止发送。

Pipeline 中有量方式可以解除发送方的阻塞，一是发送方创建充足空间的 channel 来发送数据，二是当接收方停止接收数据时，明确通知发送方。

## 摘要树

一个真实的案例。

MD5，消息摘要算法，可用于文件校验和的计算。下面的输出是命令行工具 md5sum 输出的文件摘要信息。

```bash
$ md5sum *.go
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

我们的例子和 md5sum 类似，不同的是，传递给这个程序的参数是一个目录。程序的输出是目录下每个文件的摘要值，输出的顺序按文件名排序。

```bash
$ go run serial.go .
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

主函数，第一步调用 MD5All，它返回的是一个以文件名为 key，摘要值为 value 的 map，然后对返回结果进行排序和打印。

```go
func main() {
    // Calculate the MD5 sum of all files under the specified directory,
    // then print the results sorted by path name.
    m, err := MD5All(os.Args[1])
    if err != nil {
        fmt.Println(err)
        return
    }
    var paths []string
    for path := range m {
        paths = append(paths, path)
    }
    sort.Strings(paths)
    for _, path := range paths {
        fmt.Printf("%x  %s\n", m[path], path)
    }
}
```

MD5All 函数将是我们接下来讨论的重点。[串行版](https://blog.golang.org/pipelines/serial.go)的实现没有并发，仅仅是从文件中读取数据再计算。

```go
// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.
func MD5All(root string) (map[string][md5.Size]byte, error) {
    m := make(map[string][md5.Size]byte)
    err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if !info.Mode().IsRegular() {
            return nil
        }
        data, err := ioutil.ReadFile(path)
        if err != nil {
            return err
        }
        m[path] = md5.Sum(data)
        return nil
    })
    if err != nil {
        return nil, err
    }
    return m, nil
}
```

## 并行计算

在 [并行版](https://blog.golang.org/pipelines/parallel.go) 中，我们会把 MD5All 的计算拆分开含有两个 stage 的 pipeline。第一个 stage，sumFiles，负责遍历目录和计算文件摘要值，摘要的计算会启动一个 goroutine 来执行，计算结果将通过一个类型 result 的 channel 发出。

```go
type result struct {
    path string
    sum  [md5.Size]byte
    err  error
}
```

sumFiles 返回了 2 个 channel，一个用于接收计算的结果，一个用于接收 filepath.Walk 的 err 返回。walk 会为每个文件启动一个 goroutine 执行摘要计算和检查 done。如果 done 关闭，walk 将立刻停止。

```go
func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
    // For each regular file, start a goroutine that sums the file and sends
    // the result on c.  Send the result of the walk on errc.
    c := make(chan result)
    errc := make(chan error, 1)
    go func() {
        var wg sync.WaitGroup
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            wg.Add(1)
            go func() {
                data, err := ioutil.ReadFile(path)
                select {
                case c <- result{path, md5.Sum(data), err}:
                case <-done:
                }
                wg.Done()
            }()
            // Abort the walk if done is closed.
            select {
            case <-done:
                return errors.New("walk canceled")
            default:
                return nil
            }
        })
        // Walk has returned, so all calls to wg.Add are done.  Start a
        // goroutine to close c once all the sends are done.
        go func() {
            wg.Wait()
            close(c)
        }()
        // No select needed here, since errc is buffered.
        // 不需要使用 select，因为 errc 是带有 buffer 的 channel
        errc <- err
    }()
    return c, errc
}
```

MD5All 将从 c channel 中接收计算的结果，如果发生错误，将通过 defer 关闭 done。

```go
func MD5All(root string) (map[string][md5.Size]byte, error) {
    // MD5All closes the done channel when it returns; it may do so before
    // receiving all the values from c and errc.
    done := make(chan struct{})
    defer close(done)

    c, errc := sumFiles(done, root)

    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```

## 并行限制

在 [并行版本](https://blog.golang.org/pipelines/parallel.go) 中，MD5All 为每个文件启动了一个 goroutine。但如果一个目录中文件太多，这可能会导致分配的内存过大以至于超过了当前机器的限制。

我们可以通过限制并行读取的文件数，限制内存分配。在 [并发限制版本](https://blog.golang.org/pipelines/bounded.go)中，我们创建了固定数量的 goroutine 读取文件。现在，我们的 pipeline 涉及 3 个 stage：遍历目录、文件读取与摘要计算、结果收集。

第一个 stage，遍历目录并通过 paths channel 发出文件。

```go
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
    paths := make(chan string)
    errc := make(chan error, 1)
    go func() {
        // Close the paths channel after Walk returns.
        defer close(paths)
        // No select needed for this send, since errc is buffered.
        errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            select {
            case paths <- path:
            case <-done:
                return errors.New("walk canceled")
            }
            return nil
        })
    }()
    return paths, errc
}
```

第二个 stage，启动固定数量的 goroutine，从 paths channel 中读取文件名称，处理结果发送到 c channel。

```go
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
    for path := range paths {
        data, err := ioutil.ReadFile(path)
        select {
        case c <- result{path, md5.Sum(data), err}:
        case <-done:
            return
        }
    }
}
```

和之前的例子不同，digester 将不会关闭 c channel，因为多个 goroutine 共享这个 channel，计算结果都将发给这个 channel 上。

相应地，MD5All 会负责在所有摘要完成后关闭这个 c channel。

```go
    // Start a fixed number of goroutines to read and digest files.
    c := make(chan result)
    var wg sync.WaitGroup
    const numDigesters = 20
    wg.Add(numDigesters)
    for i := 0; i < numDigesters; i++ {
        go func() {
            digester(done, paths, c)
            wg.Done()
        }()
    }
    go func() {
        wg.Wait()
        close(c)
    }()
```

我们也可以为每个 digester 创建一个单独的 channel，通过自己的 channel 传输结果。但这种方式，我们还要再启动一个新的 goroutine 合并结果。

最后一个 stage，负责从 c 中接收处理结果，通过 errc 检查是否有错误发生。该检查无法提前进行，因为提前执行将会阻塞 walkFile 往下游发送数据。

```go
    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    // Check whether the Walk failed.
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```

## 总结

这篇文章介绍在 Go 中如何正确地构建流式数据 pipeline。

它的异常处理非常复杂，pipeline 中的每个 stage 都可能导致上游阻塞，而下游可能不再关心接下来的数据。关闭 channel 可以给所有运行中的 goroutine 发送 done 信号，这能帮助我们成功解除阻塞。如何正确地构建一条流式数据 pipeline，文中也总结了一些指导建议。

我的博文：[Go 如何构建并发 Pipeline](https://www.poloxue.com/posts/2019-07-05-golang-pipeline)，译：[Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)

