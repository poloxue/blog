---
title: "Go 中如何实现 WebSocket？从原理到第三方库"
date: 2024-02-11T18:43:12+08:00
draft: true
comment: true
description: "我们知道，基于 HTTP 轮询的通信模式不仅仅是浪费服务器资源，而且还降低了用户体验。这种情况，直到 2008 年 WebSocket 的出现，才有所改变。WebSocket 提供实时、双向通信，极大地提高了交互性和效率。"
---

我们知道，在传统的 HTTP 通信模式下，客户端必须通过轮询的方式检查服务器上是否有新信息。但这种方式有明显的缺点：会产生大量无用的请求，增加服务器负担，且反应速度慢，降低了用户体验。

直到 2008 年 WebSocket 的出现，这种情况才有所改变。

WebSocket 提供实时、双向通信，极大地提高了交互性和效率。而且，2009 年，Go 语言被开发出来，它有着优秀的并发特性，和 WebSocket 配合，天然为构建实时通信应用提供了更多可能。

那让我们今天就来聊聊这个主题，从 WebSocket 以及如何在 Go 中使用它。

## WebSocket 是什么？

WebSocket 协议是一种在单个TCP连接上进行全双工通信的网络协议，它允许服务器和客户端之间进行双向实时通信。WebSocket协议在 2011 年成为国际标准，基本所有浏览都对它提供了支持。它是实现网页或应用中实时通信功能的关键技术之一。

全双工通信：WebSocket允许数据同时在两个方向上流动，即服务器可以随时向客户端发送消息，客户端也可以随时向服务器发送消息。

建立连接：WebSocket 连接的建立依赖 HTTP 协议。Client 发送一个特殊的HTTP请求（包含 Upgrade 头）到服务器来发起 WebSocket 连接。如果服务器支持WebSocket，它会返回一个应答响应，确认连接的升级。这个过程称为“握手”。握手完成后，连接就从 HTTP 升级到WebSocket协议。

数据帧：WebSocket 数据是通过帧进行传输的，这些帧可以包含文本或二进制数据。这使得 WebSocket 非常适合传输各种类型的数据，从简单的消息到复杂的二进制对象，如图片或视频流。

与 HTTP 轮询方式相比，WebSocket 提供了真正的实时通信能力。一旦建立了 WebSocket 连接，数据可以无需等待客户端的请求就立即从服务器推送到客户端，反之亦然。这减少了延迟，提高了通信效率，并减少了服务器资源的消耗。

## WebSocket 的数据帧

待补充！

## Go 与 WebSocket

Go 的并发主要是围绕 goroutine 和 channel 构建的，非常适合开发如 websocket 这类需要高并发处理能力的应用。

我们可将每个 WebSocket 连接放到一个goroutine 中处理，这样即使某个连接的数据传输被阻塞，也不会影响到其他连接。

而对于不同 WebSocket 连接间的数据共享，则非常适合通过 channel 进行传递。

## Go 中的 WebSocket 库

在 Go 的生态系统里，WebSocket 相关的库有几个。而最著名和使用广泛使用的是 gorilla/websocket，它被广泛认为是 Go 中最稳定和功能丰富的WebSocket库，支持 WebSocket 协议中的各种帧类型，如控制帧（ping/pong/close）和数据帧（text/binary）。而且，它还允许对 WebSocket 连接的细粒度配置，以满足不同场景需求。

Go 中其他比较出名的 Websocket 库还有如 noghr.io/websocket

### 控制帧（Ping/Pong/Close）的处理

*Ping/Pong消息**：`gorilla/websocket`自动处理Ping和Pong控制消息。当库接收到Ping消息时，会自动回复Pong消息。此行为确保了连接的活性，无需开发者手动干预。如果需要在接收到Ping或Pong消息时执行特定的操作，可以设置对应的处理器：
```go
conn.SetPingHandler(func(appData string) error {
    // 处理接收到的Ping消息
    return nil
})
  
conn.SetPongHandler(func(appData string) error {
    // 处理接收到的Pong消息
    return nil
})
```

**Close消息**：当对端发送Close消息时，`gorilla/websocket`会自动回复Close消息以完成关闭握手。开发者可以通过`CloseHandler`自定义关闭过程中的逻辑：
```go
conn.SetCloseHandler(func(code int, text string) error {
    // 自定义处理关闭逻辑
    return nil
})
```

### 数据帧（Text/Binary）的处理

**发送和接收文本消息**：使用`WriteMessage`方法发送文本消息，使用`ReadMessage`或`NextReader`方法接收消息。
```go
err := conn.WriteMessage(websocket.TextMessage, []byte("Hello"))

_, message, err := conn.ReadMessage()
```

**发送和接收二进制消息**：类似于文本消息，但在调用`WriteMessage`方法时指定消息类型为`websocket.BinaryMessage`。

```go
err := conn.WriteMessage(websocket.BinaryMessage, binaryData)
```

### 连接的细粒度配置

`gorilla/websocket`提供了多种方式来配置WebSocket连接，包括调整缓冲区大小、设置读写超时、自定义HTTP头等。

**读写缓冲区大小**：在创建连接时，可以通过`Upgrader`结构体调整默认的I/O缓冲区大小：
```go
var upgrader = websocket.Upgrader{
    ReadBufferSize:  1024,
    WriteBufferSize: 1024,
}
```

**调整握手过程中的参数**：`Upgrader`还允许开发者在握手过程中设置诸如检查来源、自定义响应头等参数，以支持更复杂的场景：

```go
upgrader.CheckOrigin = func(r *http.Request) bool {
    // 检查来源逻辑
    return true
}
```

**设置读写超时**：为防止慢连接或死连接，可以为读操作和写操作设置超时：
```go
conn.SetReadDeadline(time.Now().Add(readTimeout))
conn.SetWriteDeadline(time.Now().Add(writeTimeout))
```

这些配置项为我们提供了强大灵活的提升 websocket 通信质量的能力。

## 实现 WebSocket 服务器

- 详细介绍如何使用 Go 语言和选定的库创建一个基本的 WebSocket 服务器。
  - 设置项目和安装依赖。
  - 编写 WebSocket 服务器代码，包括升级 HTTP 连接到 WebSocket、接收和发送消息等。
  - 解释如何处理并发连接，包括使用 goroutine 管理每个 WebSocket 连接。

## 实现 WebSocket 客户端

- 展示如何创建一个简单的 WebSocket 客户端（可以是 Web 前端使用 JavaScript，也可以是 Go 客户端）。
  - 对于 Web 客户端，展示如何使用原生 WebSocket API。
  - 对于 Go 客户端，展示如何使用相同或不同的库连接到 WebSocket 服务器。

## 6. 实战案例：构建一个实时聊天应用

- 通过构建一个实时聊天应用来整合前面的概念。
  - 服务器端：处理多个客户端连接，广播消息给所有连接的客户端。
  - 客户端：发送消息到服务器并接收来自服务器的广播消息。

## 7. 安全和性能优化

- 讨论 WebSocket 安全性问题，如何加密 WebSocket 连接（wss://）。
- 提供性能优化建议，比如如何有效管理大量的并发连接，连接保活（心跳检测）等。

### 8. 结论
- 总结 WebSocket 和 Go 语言的强大组合，以及它们如何能够帮助开发者构建高效的实时通信应用。
- 鼓励读者动手实践，探索更多的应用场景。

