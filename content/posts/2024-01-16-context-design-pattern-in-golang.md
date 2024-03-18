---
title: "从 Context 看 Go 设计模式：接口、封装和并发控制"
date: 2024-01-16T20:10:26+08:00
draft: false
comment: true
description: "在 Go 语言中，`context` 包是并发编程的核心，用于传递取消信号和请求范围的值。但其传值机制，特别是为什么不通过指针传递，而是通过接口。虽然是简单问题，但值得引发我的一些思考。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-16-context-design-pattern-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 小技巧第五篇，系列文章查看：[Go 语言小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

在 Go 语言中，`context` 包是并发编程的核心，用于传递取消信号和请求范围的值。但其传值机制，特别是为什么不通过指针传递，而是通过接口。虽然是简单问题，但值得引发我的一些思考。

考虑以下典型的代码片段：

```go
package main

import "context"

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    // ... call cancel() when specified signals are triggered

    handle(ctx)
}

func handle(ctx context.Context) error {
    return nil
}
```

这段代码展示了在 Go 中创建和传递 `context` 的简单用法。但背后的设计理念和实现细节却值得研究。

为什么 `context` 是以接口的形式传递，而非指针？这不仅涉及 Go 的并发哲学，还关系到封装性、并发安全性和接口的灵活性。

本文将简要探讨 `context` 包的设计和实现，着重解析其非指针传值的原因，从而揭示 Go 并发模型背后的设计智慧。

## Context 的基本结构

首先，如上的代码中，通过 `context.WithCancel(context.Background())` 返回的是一个 `context.Context` 类型，而需要明确的是，`context.Context` 是一个接口，而不是一个具体的数据结构。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-16-context-design-pattern-in-golang-02.png)


这个接口定义了四个方法：Deadline、Done、Err 和 Value。这些方法提供了控制和访问 context 的手段。

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

## Context 的实现和传递机制

在 Go 中，`context` 的实现是通过结构体和指针的组合完成。例如，`WithCancel` 函数返回的 `context.Context` 类型实际上是一个指向 cancelCtx 结构体的指针。

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}
```

这里的关键在于理解 Go 语言的传值机制。

在 Go 中，所有的函数参数都是通过值传递的。这意味着传递给函数的总是数据的副本，而不是数据本身。

然而，当你传递一个指针时，你传递的是指针的副本，这个副本指向原始数据。因此，即使 Go 语言只有值传递，你仍然可以通过传递指针的副本来修改原始数据。

在调用方，我们拿到 `WithCancel` 返回的指针，因为它的内部实现，满足 `context.Context` 的接口约束，能成功转为 `Context` 接口类型。
## 为什么 Context 不直接传递指针

虽然 `context` 的某些实现（如 `cancelCtx`）在内部使用指针，但 `context.Context` 接口本身并不暴露任何指针。

为什么要这么做呢？

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-16-context-design-pattern-in-golang-03.png)

**封装性**

通过将具体结构隐藏在接口后面，`context` 包确保了用户不能直接访问或修改内部状态，这是良好封装的标志。如其中的 `timerCtx` 保存了时间信息，而 `valueCtx` 保存了请求范围了的上下文信息，这些数据保证一致性和并发安全。

这种设计防止了不恰当的使用，保持了 context 的行为一致性和预测性。

**并发安全**

context 被设计为并发安全的。如果 context 通过指针传递，暴露内部实现，那么在并发访问时，可能就有方式修改实际数据的内部状态。

通过接口隐藏实现细节，context 的设计者可以确保内部状态的同步和一致性，而不需要用户介入。

**灵活性**

`context.Context` 是一个接口，这意味着你可以有多种实现。

如果 context 通过指针传递，那么所有的实现都必须是具体的结构体，如 handle 函数指定传递 `cancelCtx` 的话，那就不能传递 `timerCtx`、`valueCtx` 等其他类型 `Context` 实现类。而通过使用接口，Go 语言允许更多的灵活性和实现多样性。

我们已经完成了 `context` 包设计理念的探讨，尤其是它如何通过接口和封装来保证并发安全性，同时提供清晰的抽象。

最后，让我们通过一个具体的例子来展示 Go 语言的这种设计模型。
## 案例：DataStore

我们要实现一个 `datastore` 实现存储数据，要求同时提供两种版本的实现：并发安全和无并发安全版本。

代码片段如下所示，它展示了 `DataStore` 接口的两种不同实现：`safeDataStore` 和 `inMemoryDataStore`。

`DataStore` 是一个接口，定义了对数据的操作。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-16-context-design-pattern-in-golang-04.png)

`DataStore` 是一个接口的具体代码，如下所示：
```go
type DataStore interface {
	ReadData() string
	WriteData(data string)
}
```

`safeDataStore` 是一个实现了 `DataStore` 接口的结构体，它使用 `sync.Mutex` 来保证并发安全.
```go
type safeDataStore struct {
	mu   sync.Mutex
	data string
}

func (ds *safeDataStore) ReadData() string {
	ds.mu.Lock()
	defer ds.mu.Unlock()
	return ds.data
}

func (ds *safeDataStore) WriteData(data string) {
	ds.mu.Lock()
	defer ds.mu.Unlock()
	ds.data = data
}
```

`inMemoryDataStore` 是另一个实现了 `DataStore` 接口的结构体，它假设只在单个 `goroutine` 中使用，因此不需要同步机制
```go

type inMemoryDataStore struct {
	data string
}

func (ds *inMemoryDataStore) ReadData() string {
	return ds.data
}

func (ds *inMemoryDataStore) WriteData(data string) {
	ds.data = data
}
```

如上的代码所示，`DataStore` 接口定义了数据存储的基本操作。同时，我们提供了两种实现：
- `safeDataStore` 使用互斥锁来保证并发安全，适用于并发场景；
- `inMemoryDataStore` 只在单 `goroutine` 中使用，不涉及任何同步机制，适用于简单场景。


使用这两个实现的代码如下：
```go
func main() {
	var store DataStore

	// 使用 safeDataStore
	store = &safeDataStore{}
	store.WriteData("safe data")
	fmt.Println(store.ReadData())

	// 使用 inMemoryDataStore
	store = &inMemoryDataStore{}
	store.WriteData("in-memory data")
	fmt.Println(store.ReadData())
}
```



通过这个例子，可以看到 Go 语言是如何通过这种模式来支持多样性和灵活性的。不同的 `DataStore` 实现可以有不同的内部行为和性能特性，但它们对外提供了统一接口。这种设计不仅使代码模块化和易于维护，而且更加易于扩展性。

## 结论

总结来说，无论是在 `context` 包的设计中，还是在我们的 `DataStore` 示例中，Go 语言的接口和封装都展现了其在构建并发安全且易于维护的系统方面的强大能力。通过这些机制，Go 语言为开发者提供了一种清晰、一致且灵活的方式来管理和传递程序的状态和行为。

博文地址：[从 Context 看 Go 设计模式：接口、封装和并发控制](https://www.poloxue.com/posts/2024-01-16-context-design-pattern-in-golang/)
