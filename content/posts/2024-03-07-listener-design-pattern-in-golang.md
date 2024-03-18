---
title: "Go 中的监视器模式与配置热更新"
date: 2024-03-09T08:00:00+08:00
draft: false
comment: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-07-listener-design-pattern-in-golang-01.png)

上篇介绍 GO 的 GUI 库 Fyne 时，提到 Fyne 的数据绑定用到了监听器模式。本文就展开说下我对 Go 中监听器模式的理解和应用吧。

## 监听器模式简介

监听器模式，或称观察者模式，它主要涉及两个组件：主题（Subject）和监听器（Listener）。

Subject 负责维护一系列的监听器，在所观测主题状态变化，将这个事件通知给所有注册的监听器。我统一将其定义为注册中心 Registry。而监听器 Listener 则是实现了特定接口的对象，用于响应事件消息，执行处理逻辑。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-07-listener-design-pattern-in-golang-02.png)

对具体应用而言，通常还会分出一个 `Watcher` 或者 `Monitor` 用于检测变化并推送给 `Registry`。从而实现将检测目标从系统解耦，无视监控组件类别。

这个模式在组件之间建立一种松散耦合的关系。将特定事件通知到关心它的其他组件，无需它们直接相互引用。看起来这个不也是发布-订阅模式吗？差不多一个意思。

之前工作中，用它最多的是配置的热更新场景，这篇文章也会简单介绍基于它的 ETCD 配置热更新。

## Go 实现监听器模式

如何用 Go 实现监听模式？我将定义两个新类型分别是注册中心（Registry）和监听器接口（Listener）。

```go
type Listener interface {
    OnTrigger()
}
```

首先是 `Listener`，它是一个接口，用于实现事件的响应逻辑。它的实现类型要求支持 `OnTrigger` 方法，会在事件发生时被执行。

```go
type Registry struct {
    listeners []Listener
}

func (r *Registry) Register(l Listener) {
    r.listeners = append(r.listeners, l)
}

func (r *Registry) NotifyAll() {
    for _, listener := range r.listeners {
        listener.OnTrigger(key, value)
    }
}
```

`Registry` 是所有监听器的注册地，当特定事件发生，我们通过 `Registry.NotifyAll` 将事件传递给所有 `Listener`。

让我们实现一个简单的案例，当监听到某个事件发生，打印 "A specified event accured"。本案例的事件将先通过主函数调用 `NotifyAll` 模拟触发事件。

为了打印事件发生消息。创建新类型实现 `Listener`：

```go
type EventPrinter struct {
}

func (printer *EventPrinter) OnTrigger() {
  fmt.Println("A specified event accured!")
}
```

写个主函数测试下是否符合预期，代码如下所示：

```go
func main() {
  r := &Registry{}
  r.Reigster(&EventPrinter{})

  // 模拟接收到消息，触发事件通知
  r.NotifyAll()
}
```

让我们执行测试，内容如下所示：

```bash
$ go run main.go
A specified event occurred
```

如果希望自定义接受函数，只需在 `Listener` 支持自定义事件接收函数即可。

```go
type EventHandler struct {
  callback func()
}

func NewEventHandler(callback func()) *EventHandler {
  return &EventHandler{callback: callback}
}
func (e *EventHandler) OnTrigger() {
  e.callback()
}
```

注册一个 `EventHandler` 到 `Registry`，主函数代码：

```go
func main() {
  r := &Registry{}
  r.Reigster(&EventPrinter{})
  r.Reigster(NewEventHandler(func() {
    fmt.Println("Custom Print: a specified event occurred!")  
  }))
  r.NotifyAll()
}
```

测试执行：

```bash
$ go run main.go
A specified event occurred!
Custom Print: a specified event occurred!
```

## 基于 Go Channel 实现并发处理

前面的示例中 `NotifyAll` 是通过 for 循环依次调用 `listener.OnTrigger` 将消息发送给 `Listener`，处理效率低下。如何加速呢？最直接的方法是通过 `goroutine` 运行 `listener.OnTrigger` 方法。

```go
func (r *Registry) NotifyAll() {
  for _, listener := range r.listeners {
      go listener.OnTrigger()
  }
}
```

还有一种方法，通过 Channel 传递事件消息，这样每个 `Listener` 有独立的 goroutine 监听和处理。

如下是 `Listener` 的实现代码：

```go
type Listener struct {
	EventChannel chan struct{}
	Callback     func()
}

func NewListener(callback func()) *Listener {
	return &Listener{
		EventChannel: make(chan struct{}, 1), // 带缓冲的 channel，防止阻塞
		Callback:     callback,
	}
}

func (l *Listener) Start() {
	go func() {
		for range l.EventChannel {
			l.Callback()
		}
	}()
}
```

这里 `Listener` 的事件处理函数在单独的 goroutine 中运行。而相应的 Registry 实现也需要修改，代码变更如下所示：

```go
type Registry struct {
	listeners []*Listener
}

func (r *Registry) Register(listener *Listener) {
	r.listeners = append(r.listeners, listener)
	listener.Start() // 启动监听器的 goroutine
}

func (r *Registry) NotifyAll(message string) {
	for _, listener := range r.listeners {
		listener.EventChannel <- struct{}{} // 发送事件到监听器
	}
}

func (r *Registry) Close() {
	for _, listener := range r.listeners {
		close(listener.EventChannel) // 关闭 channel，停止监听器 goroutine
	}
}
```

整体上的变化不大，在 `listner.Register` 方法中启动 `Listener` 事件处理 goroutine 等待事件消息。

## 实际案例：ETCD 配置热更新

让我们实践一个具体的应用场景：实现配置的动态更新以及组件的自动重连机制。

我们将针对包括 MySQL、Redis 在内的各种组件，实现它们在配置变更时能够自动重连。这些组件的配置信息将以 JSON 格式存储于 ETCD 的多个键（Key）中。

假设，配置结构如下所示：

```go
type MySQLConfig struct {
  Host     string
  Port     int
  User     string
  Password string
}

type RedisConfig struct {
  Host     string
  Port     int
}
```

这些配置被保存在 ETCD 中，我们要实时监控配置的变化并据此更新配置和执行重连操作。

示例用法如下所示：

```go
registry.Register("/config/mysql", func(data) {
  // unmarshal data
  // reconnect mysql
})
```

让我们基于监听器模式简单设计一个模块，实现 ETCD 热更新：

- 每个监听器可以订阅特定的 key 或 key 前缀的更新事件。
- 使用 `channel` 通知配置变更，触发对应的监听回调。

这个示例，函数回调和轮询其实已经满足需求，此处只是为了演示，而是否使用 channel 要具体分析。

我们这个设计要涉及到三个部分。分别是 Watcher、Listener 和 Registry。

- Watcher 责监听 ETCD 中的 key 变更事件。
- Listener 定义了当特定 key 发生变化时需要执行的回调逻辑。
- Registry 管理所有 Listener，将 ETCD 变更事件分发给对应 Listener。


先定义 `Event` 类型，一个简单的结构体，表示 ETCD 中 key 的变更事件：

```go
type Event struct {
  Key   string
  Value string
}
```

### Listener

`Listener` 实现如下所示：

```go
type Listener struct {
	EventChannel chan *Event
	Callback     func(*Event)
}

func NewListner(callback func(*Event)) *Listener {
	l := &Listener{
		EventChannel: make(chan *Event),
		Callback:     callback,
	}
	return l
}

func (l *Listener) Start() {
	go func() {
		for event := range l.EventChannel {
			l.Callback(event)
		}
	}()
}
```

基本之前的没太大差别，从 `EventChannel` 中拿到事件消息，调用回调函数。

### 实现 Registry

`Registry` 负责维护 `Listener` 的注册，并在接收到 key 变更事件时通知相关的 `Listener`：

```go
type Registry struct {
	listeners map[string][]*Listener
}

func NewRegistry() *Registry {
	return &Registry{
		listeners: make(map[string][]*Listener),
	}
}

func (r *Registry) Register(key string, listener *Listener) {
	r.listeners[key] = append(r.listeners[key], listener)
	listener.Start()
}

func (r *Registry) Notify(event *Event) {
	if listeners, ok := r.listeners[event.Key]; ok {
		for _, listener := range listeners {
			listener.EventChannel <- event
		}
	}
}
```

注册 `Listener` 到 `Registry` 中，通过 `map` 将 `key` 与 `Listener` 关联起来。

### 实现 Watcher

`Watcher` 负责从 ETCD 订阅 key 的变更事件，并将这些事件发送到 `Registry` 的 `eventChannel` 上：

```go
func WatchEtcdKeys(client *clientv3.Client, registry *Registry, watchKeys ...string) {
  for _, key := range watchKeys {
    go func(key string) {
      watchChan := client.Watch(context.Background(), key, clientv3.WithPrefix())
      for wresp := range watchChan {
        for _, ev := range wresp.Events {
          event := &Event{
            Key:   string(ev.Kv.Key),
            Value: string(ev.Kv.Value),
          }
          registry.Notify(event)
        }
      }
    }(key)
  }
}
```

### 使用示例

让我们实际在 main 函数上使用一下，观察行为是否正常。

```go
func main() {
  client, err := clientv3.New(clientv3.Config{
      Endpoints:   []string{"localhost:2379"},
  })
  if err != nil {
      log.Fatal(err)
  }
  defer client.Close()

  registry := NewRegistry()
    // 注册监听器
  registry.Register("/config/mysql", NewListener(func(event * Event) {
    fmt.Println(event)
    // 执行数据重连之类的操作
  }))

    // 开始监听 ETCD key 变更
  WatchEtcdKeys(client, registry, "/config/")
  time.Sleep(10 * time.Minute)
}
```

这个示例创建了一个 ETCD 客户端，初始化了一个 `Registry`，并为特定的 key 注册了一个 `Listener`。然后，通过 `WatchEtcdKeys` 函数开始监听 `/config/` 前缀下的所有 key 的变更。

这种设计支持对特定 key 或 key 前缀的监听。当相关 key 变更时，通过 `channel` 通知 `Listener`，而收到更新事件后的具体操作。视场景而定，这里是执行重连操作。

特别说明，示例仅作为概念验证，实际应用中需要更多的错误处理和优化。
