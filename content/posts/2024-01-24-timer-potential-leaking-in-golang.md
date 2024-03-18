---
title: "Go 定时器：如何避免潜在的内存泄漏陷阱"
date: 2024-01-24T13:28:13+08:00
draft: false
comment: true
description: "这篇文章将探讨的是 Go 中如何高效使用 timer，特别是与select 一起使用时，如何防止潜在的内存泄漏问题。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-24-timer-potential-leaking-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 小技巧第七篇，系列文章查看：[Go 语言小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

这篇文章将探讨的是 Go 中如何高效使用 timer，特别是与select 一起使用时，如何防止潜在的内存泄漏问题。

## 引出问题

先看一个例子，我们在 Go 中的 select 使用定时器，实现为消息监听加上超时能力。

核心代码，如下所示：

```go
func main() {
  ch := make(chan int)
  // 启动一个goroutine
  go func() {
    for {
      select {
      case num := <-ch:
        fmt.Println("获取到的数字是", num)
      case <-time.After(2 * time.Second):
        fmt.Println("时间到了!!!")
      }
    }
  }()
  for i := 0; i < 5; i++ {
    ch <- i
    time.Sleep(1 * time.Second)
  }
}
```

在这个例子中，select 语句用于监听 channel 消息和超时。然而，我要关注的重点是 timer 的行为。它是不是能达到我们预期的目标呢？为消息监听加上超时效果呢？

## 检查定时器行为

如果运行这段代码，将会发现，如果 timer 设置为 2 秒，主循环设置 1 秒的延迟时间，timer 不会触发。

如下是程序的运行输出：

```bash
获取到的数字是 0
获取到的数字是 1
获取到的数字是 2
获取到的数字是 3
获取到的数字是 4
```

这是因为每次循环，time.After 创建都会返回一个新的定时器，产生的后果就是，每次多会重置 select 调用的时间。

相反，如果将定时器的超时设置为 1 秒，将主循环的time.Sleep设置为 2 秒，就能触发定时器，输出 "时间到了!!!"。这证明了这个定时器是有效运行的。

## 潜在的内存泄漏

Go标准库文档提到，每次调用time.After都会创建一个新的定时器。然而，我们需要认真考虑一个重要问题。

来自官方文档引用：

> The underlying Timer is not recovered by the garbage collector until the timer fires.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-24-timer-potential-leaking-in-golang-03-v1.png)

如果这些 timer 没有达到设定时间，就不会被 GC。这会导致内存泄漏。毫无疑问，如果在常驻程序中频繁使用 timer 的，内存泄漏将会日积月累。

## 最佳实践

要高效地管理资源并避免 timer 的内存泄漏，建议使用 time.NewTimer 和 timer.Reset 组合。这种方法允许重复使用一个定时器，减少资源消耗和潜在的内存泄漏风险。


例如，如下是使用 time.NewTimer 改进的代码示例：

```go
// 为定时器定义持续时间。
idleDuration := 5 * time.Minute
// 使用指定的持续时间创建新的定时器。
idleDelay := time.NewTimer(idleDuration)
// 确保定时器适当地停止以避免资源泄漏。
defer idleDelay.Stop()
// 进入循环以处理传入的消息或基于时间的事件。
for {
  // 在每次循环迭代开始时重置定时器到指定的持续时间。
  idleDelay.Reset(idleDuration)
  
  // 使用select等待多个通道操作。
  select {
  // 处理传入消息的情况。
  case s, ok := <-in:
    // 检查通道是否关闭。如果是，退出循环。
    if !ok {
      return
    }
    // 处理接收到的消息`s`。
    // 在这里添加相关代码来处理消息。
  // 处理定时器超时的情况。
  case <-idleDelay.C:
    // 增加空闲计数器或处理超时事件。
    // 这通常是您会在这里添加代码来处理超时情况的地方。
    idleCounter.Inc()
  // 处理取消或上下文过期的情况。
  case <-ctx.Done():
    // 如果上下文已完成，则退出循环。
    return
  }
}
```

流程如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-24-timer-potential-leaking-in-golang-02-v1.png)

这里例子中演示了 Go 语言中如何正确使用和管理 timer。通过遵循 Go 标准库的建议将能产出更高效和可靠的程序。

## 结论

本文通过一个代码案例演示了 GO 中 `timer.After` 可能产生的潜在内存泄漏问题。通过使用官方推荐的方案，利用重置定时器时间实现 `Timer` 的重复利用，避免了潜在的内存泄漏问题。

博文地址：[Go 定时器：如何避免潜在的内存泄漏陷阱](https://www.poloxue.com/posts/2024-01-24-timer-potential-leaking-in-golang/)
