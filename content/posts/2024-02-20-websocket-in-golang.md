---
title: "Go 中如何实现 WebSocket？从原理到第三方库"
date: 2024-02-11T18:43:12+08:00
draft: true
comment: true
description: "我们知道，基于 HTTP 轮询的通信模式不仅仅是浪费服务器资源，而且还降低了用户体验。这种情况，直到 2008 年 WebSocket 的出现，才有所改变。WebSocket 提供实时、双向通信，极大地提高了交互性和效率。"
---

我最初只是想介绍下 Go 的三方库 gorilla/websocket 的使用，但发现如果真想把这个库讲清楚，对 websocket 要有一定深度的理解才行。于是，本文就变成重点介绍 websocket，而 gorilla/websocket 则变成了介绍 websocket 的辅助工具。

## 引言

传统的 HTTP 通信模式下，客户端必须通过轮询方式检查服务器实现近实时通信，缺点是不仅产生了大量无用的请求，增加服务器负担，而且反应速度慢，降低用户体验。

2008 年 WebSocket 的出现，这种情况才有所改变。WebSocket 是一种提供了实时、双向通信的协议。2011 年 WebSocket 被定义为国际标准（RFC6455 和 W3C)，基本被所有浏览器的支持。

## WebSocket 是什么？

WebSocket 协议是一种在单个 TCP 连接上进行全双工通信的网络协议，它允许服务器和客户端间进行双向实时通信。

全双工通信：WebSocket 允许数据同时在两个方向上流动，即服务端可以随时向客户端发送消息，客户端也可以随时向服务器发送消息。

建立连接：WebSocket 连接的建立依赖 HTTP 协议。Client 发送一个特殊的HTTP请求（包含 Upgrade 头）到服务器来发起 WebSocket 连接。如果服务器支持WebSocket，它会返回一个应答响应，确认连接的升级。这个过程称为“握手”。握手完成后，连接就从 HTTP 升级到WebSocket协议。

数据帧：WebSocket 数据是通过帧进行传输的，这些帧可以包含文本或二进制数据。这使得 WebSocket 非常适合传输各种类型的数据，从简单的消息到复杂的二进制对象，如图片或视频流。

与 HTTP 轮询方式相比，WebSocket 提供了真正的实时通信能力。一旦建立了 WebSocket 连接，数据可以无需等待客户端的请求就立即从服务器推送到客户端，反之亦然。这不仅是减少了延迟，提高了通信效率，还减低了服务器的负载。

## 案例：聊天室

使用`gorilla/websocket`库实现WebSocket通信是在Go语言开发中一种常见的做法，尤其适用于需要实时功能的应用，比如聊天室。下面将通过建立连接、收发消息和关闭连接三个步骤，逐步介绍如何使用`gorilla/websocket`实现一个简单的WebSocket聊天室。

### 1. 建立连接

#### 服务端

首先，你需要在服务端设置一个HTTP路由，用于升级HTTP连接到WebSocket连接。

```go
package main

import (
    "net/http"
    "github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
    ReadBufferSize:  1024,
    WriteBufferSize: 1024,
    // 允许所有CORS请求，实际生产中应更严格
    CheckOrigin: func(r *http.Request) bool { return true },
}

func handleConnections(w http.ResponseWriter, r *http.Request) {
    // 升级初始的GET请求到一个websocket
    ws, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Fatal(err)
    }
    // 确保我们在函数返回时关闭连接
    defer ws.Close()

    // 这里可以添加连接成功后的处理逻辑
}

func main() {
    http.HandleFunc("/ws", handleConnections)
    http.ListenAndServe(":8080", nil)
}
```

#### 客户端

在客户端，你可以使用`gorilla/websocket`库或者浏览器的原生WebSocket API发起连接。

使用JavaScript代码示例连接到WebSocket服务器：

```javascript
var socket = new WebSocket("ws://localhost:8080/ws");

socket.onopen = function(e) {
  console.log("连接服务器成功");
};

socket.onmessage = function(event) {
  console.log(`[消息] 数据来自服务器: ${event.data}`);
};

socket.onclose = function(event) {
  if (event.wasClean) {
    console.log(`[关闭] 连接已关闭，代码=${event.code} reason=${event.reason}`);
  } else {
    // 比如服务器进程被杀死或网络断开
    // 通常这是一个意外的关闭
    console.log('[关闭] 连接已关闭');
  }
};

socket.onerror = function(error) {
  console.log(`[错误] ${error.message}`);
};
```

### 2. 收发消息

#### 服务端

服务端可以使用`ws.ReadMessage()`和`ws.WriteMessage()`来读取和发送消息。

```go
for {
    messageType, message, err := ws.ReadMessage()
    if err != nil {
        log.Println("read:", err)
        break
    }
    log.Printf("recv: %s", message)

    // 广播消息到所有连接的客户端
    // 假设你有一个保存所有客户端连接的结构
    for client := range clients {
        err := client.WriteMessage(messageType, message)
        if err != nil {
            log.Println("write:", err)
            break
        }
    }
}
```

#### 客户端

客户端发送消息给服务器：

```javascript
socket.send("你好服务器");
```

### 3. 关闭连接

#### 服务端

服务端的连接会在`defer ws.Close()`语句执行时关闭，也可以根据需要主动发送关闭消息。

#### 客户端

客户端可以主动关闭连接：

```javascript
socket.close();
```

### 完整的聊天室案例

以上示例展示了使用`gorilla/websocket`库在Go中实现WebSocket通信的基本步骤。在完整的聊天室案例中，你需要维护一个客户端连接的列表，以便广播消息给所有连接的客户端。此外，还需要处理诸如连接错误、异常关闭等事件，并在必要时更新客户端列表以移除断开的连接。

这个基本框架可以根据实际需求进行扩展，例如添加用户身份验证、支持更复杂的消息格式（如JSON）、实现私聊和群聊功能等。
我将通过一个常见的聊天室案例演示 websocket 的使用，使用的是 Go 三方库 `gorilla/websocket`。目标上只要实现消息的广播即可。我重点在介绍 websocket 的整个通信过程，故而会采用通信流程展开。

## WebSocket 的数据帧

WebSocket 协议中的数据传输是基于帧的概念进行的。帧是 WebSocket 协议数据通信的最小单位。

数据帧结构如下所示：

帧起始字节：

- FIN位：占1位，表示这是否是一个消息的最后一个帧（fragment）。如果是，设置为1；如果消息由多个帧组成，则除了最后一个帧外，其他帧的FIN位都设置为0。
- RSV1、RSV2、RSV3位：各占1位，用于将来的协议扩展，当前版本中应设置为0，除非双方都理解并同意使用非0值。
- 操作码（Opcode）：占4位，定义帧的类型（如文本帧、二进制帧、关闭帧、Ping帧和Pong帧、继续帧）。

掩码和载荷长度：

- 掩码位：占 1 位，指示载荷数据是否被掩码处理。对于从客户端发送到服务器的消息，此位必须设置为1，并使用掩码键对数据进行编码。
- 载荷长度：紧跟操作码之后的7位用于表示载荷数据的长度。如果这7位不足以表示长度，则可以使用接下来的16位或64位。

掩码键（如果掩码位为1）：
- 占32位（4字节），用于掩码或解码载荷数据。每个字节的数据都与掩码键中相应的字节进行XOR操作以得到原始数据。

载荷数据：

- 数据帧的主体部分，可以包含文本或二进制数据。实际长度由前面的长度字段指定。

### 帧类型

WebSocket 协议定义了几种不同类型的帧，用于控制数据的传输：

- **文本帧（Text Frames）**：包含文本数据。当传输的是文本消息时使用此帧。
- **二进制帧（Binary Frames）**：包含二进制数据。用于传输图片、文件等二进制内容。
- **关闭帧（Close Frames）**：用于标示连接的关闭。
- **Ping帧和Pong帧**：用于心跳检测机制，维持连接的活性，Ping帧由一端发送，另一端则回复一个Pong帧。
- **继续帧（Continuation Frames）**：允许发送开始帧后被分段的消息的后续部分。

### 数据传输

在 WebSocket 连接中，数据通过一系列帧进行传输。每个帧都包括了描述数据类型（如文本或二进制）的标识符，以及负载数据本身。这种结构化的传输方式让WebSocket非常灵活和高效，能够满足多种实时通信场景的需求。

- **消息分片**：较大的消息可以分成多个帧进行发送，接收方将这些帧重新组合以重建原始消息。这允许WebSocket处理大量数据而不会影响到连接的稳定性和性能。
- **控制消息**：WebSocket通过特定的控制帧（如Ping/Pong帧）实现了连接的实时监测和管理，确保连接的活跃性并允许及时地进行错误处理或连接关闭。

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

## 安全和性能优化

- 讨论 WebSocket 安全性问题，如何加密 WebSocket 连接（wss://）。
- 提供性能优化建议，比如如何有效管理大量的并发连接，连接保活（心跳检测）等。

## 8. 结论

- 总结 WebSocket 和 Go 语言的强大组合，以及它们如何能够帮助开发者构建高效的实时通信应用。
- 鼓励读者动手实践，探索更多的应用场景。

