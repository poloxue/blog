---
title: "从 fatal 错误到 sync.Map：Go中 Map 的并发策略"
date: 2024-01-21T15:34:16+08:00
draft: false
comment: true
description: "为什么 Go 语言在多个 goroutine 同时访问和修改同一个 map 时，会报出fatal错误而不是panic？这篇文章将带你一步步了解背后的原理，并引出解决 map 并发问题的方案。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-02.png)

> 嗨，大家好！本文是系列文章 Go 小技巧第二篇，系列文章查看：[Go 语言小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

为什么 Go 语言在多个 goroutine 同时访问和修改同一个 map 时，会报出 fatal 错误而不是 panic？我们该如何应对 map 的数据竞争问题呢？

这篇文章将带你一步步了解背后的原理，并引出解决 map 并发问题的方案。

## Map 数据竞争

首先，什么是 Map 数据竞争。

当两个或多个 goroutine 在没有适当同步机制的情况下，同时访问同一块数据，且至少有一个 goroutine 在修改这块数据，就会发生数据竞争。这种情况可能导致程序的行为异常，甚至崩溃。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-01.png)

而 map 是 Go 中的一种常用的数据结构，提供了快速的 Key/Value 存储能力。但 Go 默认的 map 并不提供并发安全。这意味着，如果我们没有采取措施来控制 map 同步访问，如果多个 goroutine 同时对一个map进行读写操作，就可能会引发数据竞争。

## Map 数据竞争产生 fatal error

在 Go 语言中，处理错误的方式通常是通过返回 Error 或者 panic。然而一旦程序检测到 map 的数据竞争，就会抛出 fatal 错误。而 fatal error 即意味会立刻崩溃。毫无疑问，Go 选择了更严格的处理方式。

通过一个简单的例子演示 fatal 错误是如何被触发的：

```go
package main

import (
    "sync"
)

func main() {
    m := make(map[int]int)
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                m[j] = j
            }
        }()
    }
    wg.Wait()
}
```

这个例子中，我创建了一个map 类型变量 `m`，然后启动了10个 goroutine，每个 goroutine 都尝试向map中写入 1000 个键值对。由于 map 在 Go 中不是并发安全的，这将导致数据竞争。

这个代码可能会触发如下的 fatal 错误，输出如下所示：

```bash
fatal error: concurrent map writes

goroutine 6 [running]:

... 省略

exit status 2
```

## 为什么 fatal 错误, 而非 panic?

fatal 错误让我们没有在程序运行时进行补救。我猜测这背后的原因主要是以下两点：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-03.png)

**立即暴露问题**

这种处理方式确保了一旦发生数据竞争，程序将立即停止运行，迫使我们直面问题，不能逃避。

这不仅有利于我们快速发现解决并发 bug，也促使我们编写代码时，应更注重并发安全，避免发生这类问题。

虽然，这种方式可能会导致程序在运行时突然停止，但长远来看，它有助于提高程序的健壮性和可靠性。

**防止数据腐败**

数据竞争的后果可能非常严重，尤其是在复杂的并发系统中。

当多个goroutine不协调地访问和修改同一个map时，可能会导致map的内部状态变得不一致，甚至损坏。

这种状态的不确定性不仅会导致程序行为异常，还可能导致难以追踪的bug。

## 深入源码：map 并发检测

当 Go 检测到 map 的并发写入，会通过 `throw` 函数抛出 fatal 错误。这一过程发生在 `mapassign` 函数中。

以下是简化后的 `mapassign` 函数的伪代码，：

```go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    // 检查是否有其他goroutine正在写入map
    if h.flags&hashWriting != 0 {
        throw("concurrent map writes")
    }
    
    // ...其他map赋值逻辑...
    
    // 设置标志位，表示有goroutine正在写入map
    h.flags |= hashWriting
    
    // ...执行map的赋值逻辑...
    
    // 写入完成，清除写入标志位
    h.flags &^= hashWriting
    
    return val
}
```

通过这段代码，就能理解 fatal 错误是如何被触发的。对，重点就是 `h.flags&hashWriting ` 这段条件判断。

## 如何避免数据竞争

在 Go 中，最常用的并发控制机制是使用 channel 或 sync 包中的工具。另外，Go 还提供了一个并发安全的 map 类型 - sync.Map。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-04.png)

### sync.Mutex

我将通过以下这段代码演示如何使用 sync.Mutex 避免数据竞争：

```go
package main

import (
    "sync"
)

func main() {
    m := make(map[int]int)
    var mu sync.Mutex
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                mu.Lock()
                m[j] = j
                mu.Unlock()
            }
        }()
    }
    wg.Wait()
}
```

在这个例子中，我们使用了 `sync.Mutex` 确保每次只有一个 goroutine 可以写入 map，从而避免数据竞争，保证程序的稳定性和正确性。

### sync.Map

虽然 Go 的标准 map 非并发安全，但 Go 在 1.9 版本中引入了一个并发安全的 map 类型 - sync.Map，专门设计来处理并发场景下的 Key/Value 存储。

sync.Map 有一些特别的特性，它不需要显式的锁操作来保证并发安全，为它内部已经处理好了同步机制，这可简化我们的并发编程。

以下示例使用 sync.Map 改写了之前通过 sync.Mutex 实现的代码。

```go
package main

import (
    "sync"
    "fmt"
)

func main() {
    var sm sync.Map
    wg := sync.WaitGroup{}

    // 写入数据
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            sm.Store(n, n*n)
        }(i)
    }

    // 读取数据
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            if value, ok := sm.Load(n); ok {
                fmt.Printf("Key: %v, Value: %v\n", n, value)
            }
        }(i)
    }

    wg.Wait()
}
```

这个例子中，sync.Map 的 Store 方法用于存储 Key/Value，Load 方法用于读取数据。我们通过这种方式就可以在多个 goroutine 中安全地使用map，而不必担心数据竞争。

另外，sync.Map 相对于传统的 sync.Mutext + map 的组合，除简化了并发编程，它还针对并发场景做了一些优化，如无锁读、细粒度锁机制（非锁整个 Map）等等。更多细节，可自行研究。

## 总结

在本文中，我们探讨了 Go 语言处理 map 并发操作数据竞争情况下的处理方式。这种设计突显了 Go 对并发安全的重视。另外，通过 Go 提供的 sync.Mutex 和sync.Map 等工具，可有效避免数据竞争，确保我们构建出稳定和高效的并发应用。

我想说，对这些机制的理解对于我们编写出健壮的Go程序是至关重要的。

博文地址：[从 fatal 错误到 sync.Map：Go中 Map 的并发策略](https://www.poloxue.com/posts/2024-01-21-fatal-error-in-concurrent-accessing-map/)
