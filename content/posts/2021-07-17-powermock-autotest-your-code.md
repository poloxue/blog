---
title: "powermock: 一个支持 gRPC 的 Mock Server"
comment: true
date: "2021-07-17T20:33:14+08:00"
draft: false
tags: ["golang"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2021-07/2021-07-17-powermock-autotest-your-code-03.png)

> 写于 2021 年 7 月份，当时公司在推自动化测试方案，集成流水线的时候，研究了这个方案。

本文介绍的是如何基于 bilibili 的开源方案 powermock 搭建一套通用的适用于自己公司的 MockServer。

## 背景

我所在公司正处在一个高速发展的阶段，各产品线齐头并进。而我所在的部门主要负责核心能力建设与增长类业务，属于所有产品线的最下游。

业务部门希望在新产品部署产线后，我们能快速的配合。这就导致了一个非常尴尬的局面，产品确定上线时间，处在产品上线的最后阶段的我们，如果有任何异常，都可能导致上线时间被压缩。

如何防止这类情况？简单来说，就是如何防止因依赖导致项目开发的不可预期。

一个常见拉新活动的业务图为例，如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2021-07/2021-07-17-powermock-autotest-your-code-01.webp)

用户场景是，假设一名用户通过我的邀请码完成平台注册，并且完成首次购物，他才能算作我的成功邀请用户，我才能得到我想要的奖励。

一个简单的拉新活动，因为服务拆分，需要同时依赖于两个服务才可以能完成这一个活动的开发。我们的核心诉求是测试我们自己开发的邀请活动。而用户和订单服务，一方面不在测试范围之内，另一方面，这些服务是其他团队开发，在测试环境的稳定性没人保证，会成为开发排期的瓶颈，而且如果是并行开发，这些服务可能还没有完成。

如果有一个服务，能够实现依赖服务协议，方便我们尽可能的穷举依赖服务的各种场景，让我们不需要时时刻刻的依赖上游服务，是不是就能解决这个问题？

## 选型

基于这些困惑和一段时间的摸索，团队成员提出了一套新的解决思路，基于 Mock 方式解决问题。确定了这个思想，接下来就是如何实施了。

通常服务间的依赖可分两类，一类是由被依赖方主动触发的消息，二类是由依赖方主动发起的调用。消息类依赖主要容易 mock，而服务间的调用 mock 相对复杂。

当前的微服务架构下，gRPC 是主流的服务间调用的协议，Mock Server 必然需要支持，经过一番寻找，在市面上发现了最近 bilibili 的开源实现方案 powermock。

这个工具的开源看时间在 2021 年 5 份刚刚开源，powermock 同时支持 HTTP 与 gRPC 协议接口的 Mock，提供灵活的插件功能。面向对象包括前后端（HTTP、gRPC）、测试等对 Mock 有需求的所有人员。

当前这个项目的 star 为 5，顺手 star 加到了 6。虽然 star 有点少，但鉴于其特性的确是我想要的功能，肯定是要尝试一下的。powermock 最吸引我的地方在于，代码简单，易于阅读，二次开发方便。而且，对于 gRPC 的支持是一个亮点。市面上的 Mock Server 主要都是面向 HTTP 的接口，面向前端。

## 架构

为便于针对 powermock 二次开发，通过阅读源码，我整理出 powermock 的主体架构，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2021-07/2021-07-17-powermock-autotest-your-code-02.webp)

从上图可以看出， powermock 本身是一个 server，提供了 HTTP 和 gRPC 两种接入方式，即通过它可以 mock HTTP 和 gRPC 两种服务。

Mock 服务的对象是作为调用方的我们，我们希望 Mock Server 能模拟依赖服务的行为，如此才能帮助我们解决环境依赖的问题， ApiManager 提供的用于指定 Mock Server 行为规则的接口。 

插件是 powermock 扩展功能不可少的能力，通过阅读源码，主要有三类插件，分别是： 

MatchPlugin，用于解析请求到响应的匹配规则，如指定请求 id 为 1 的情况下该返回什么内容。当前支持插件有 simple （YAML 指定匹配规则） 和 script （Javascript 指定匹配规则）插件；

StoragePlugin，用于保存 Mock 规则，支持的插件有 redis 和 rediscluster 三类插件。默认规则保存在内存里，重启即丢失。redis 可永久保存规则，但没有将数据结构化存储，大规划构造测试数据不是很方便；

MockPlugin，用于生成 Mock 数据，包含三个插件，分别是有 HTTP 和 gRPC，负责将 response 按 gRPC 或者 HTTP 格式返回；

整体的架构大概如此，理解了它们，对于接下来的无论是安装配置，还是具体的使用都会有大有裨益。

## 安装

powermock 提供了 powermock 和 powermock-v8（支持 javascript 脚本解析规则） 两个命令。

安装方式如下：
```bash
$ go get github.com/bilibili-base/powermock/cmd/powermock
$ go get github.com/bilibili-base/powermock/cmd/powermock-v8 // 支持 javascript 的 server
```

## 配置

powermock 的配置分为两类，一个 mockserver 本身的配置，另外则是对于 Mock 规则的配置。

这小节介绍的内容主要是 Mock Server 自身的配置。如下是一个示例配置：

```yaml
log:
    pretty: true
    level: debug
grpcmockserver:
    enable: true
    address: 0.0.0.0:30002
    protomanager:
        protoimportpaths: [ ]
        protodir: ./apis
httpmockserver:
    enable: true
    address: 0.0.0.0:30003
apimanager:
    grpcaddress: 0.0.0.0:30000
    httpaddress: 0.0.0.0:30001
pluginregistry: { }
plugin:
    simple: { }
    grpc: { }
    http: { }
    script: { }
    redis:
        enable: false
        addr: 127.0.0.1:6379
        password: ""
        db: 0
        prefix: /powermock/
```

如果已经了解前面的架构，这里的配置就比较容易理解了。

包括：

- mock server 配置，主要是 gRPC 和 HTTP 两类服务的地址与端口配置；
- apimanager 配置，指定 grpc 和 http 的 mock API 规则管理服务的接入地址和端口；
- plugin 插件配置，插件的管理配置，这里吐槽一下， powermock 将所有类别的插件在一个 plugin 配置项下，类别区分不明晰，如果增加二级目录，如 mock、match、storage，配置会更加易懂；

## 使用教程

接下来，我将通过几个真实的案例带大家一起看下 powermock 的使用方法，如何实现 proto 和 http 服务的 Mock。

首先，创建目录 examples/，将所有的 proto 文件定义整理到这个文件下。

### 快速开始

先来介绍一个官方的快速接入的案例，介绍如何通过内存和 redis 存储 Mock 规则。

### 接口定义

首选，创建 greeter/apis/greeter.proto 文件，内容如下：

```bash

syntax = "proto3";

package examples.hello.api;
option go_package = "github.com/poloxue/mock/examples/greeter";

service Greeter {
    rpc Hello(HelloRequest) returns (HelloResponse);
}

message HelloRequest {
    string message = 2;
}

message HelloResponse {
    string message = 2;
}
```

在 greeter 目录下执行使用  protoc 命令编译 proto 文件，如下：

```bash
$ protoc -I. --go_out=paths=source_relative:. --go-grpc_out=paths=source_relative:. ./apis/*.proto
```

编译生成文件包括 greeter/apis/greeter.pb.go 和 greeter/apis/greeter_grpc.pb.go。

### 配置启动

首先是一个基本的内存版本的  mockserver，创建 greeter/config.yaml，配置如下：

```yaml
log:
  pretty: true
  level: debug
grpcmockserver:
  enable: true
  address: 0.0.0.0:30002
  protomanager:
    protoimportpaths: []
    protodir: ./apis
httpmockserver:
  enable: true
  address: 0.0.0.0:30003
apimanager:
  grpcaddress: 0.0.0.0:30000
  httpaddress: 0.0.0.0:30001
pluginregistry: { }
plugin:
  simple:
    enable: true
```

配置的具体含义参考前面的介绍，主要的是启用的 plugin 是 simple，最简单的内存版本。执行如下命令启动服务：

```bash
$ powermock serve --config.file config.yaml
9:18PM INF * start to create pluginRegistry component=main file=setup.go:40
9:18PM INF * start to create apiManager component=main file=setup.go:46
9:18PM INF * start to create httpMockServer component=main file=setup.go:63
...
9:18PM INF starting service component=main.gRPCMockServer.gRPC file=service.go:29
9:18PM INF starting service component=main.apiManager.http file=service.go:29
9:18PM INF starting service component=main.httpMockServer file=service.go:29
```

输出日志，成功启动服务。

## Mock 规则

成功启动服务之后，如何设定配置 Mock 数据的生产规则呢？即什么输入对应输出的规则。

先来一个简单的配置目标，无论输入什么都返回固定的 {"message": "hello world!"}。

创建文件 `greeter/apis.yaml`，配置如下：

```yaml
uniqueKey: "hello_example_gRPC"
path: "/examples.greeter.api.Greeter/Hello"
method: "POST"
cases:
  - response:
        simple:
        header:
          x-unit-id: "3"
          x-unit-region: "sh"
        trailer:
          x-api-version: "1.3.2"
        body: |
          {"message": "hello world!"}
```

上述不仅设置了响应的 body，还设置了 x-unit-id 等 header 和 grpc trailer 信息。

如何将规则加载到 mock server 中，命令如下：

```bash
$ powermock load --address=127.0.0.1:30000 apis.yaml
```

加载完成后，我们可以直接通过 grpcurl 命令验证下 Mock 的结果是否是我们想要的。

```bash
$ protoc --proto_path=. --descriptor_set_out=greeter.protoset --include_imports apis/*.proto
$ grpcurl -plaintext -protoset greeter.protoset 127.0.0.1:30002 examples.greeter.api.Greeter/Hello
{
  "message": "hello world!"
}
```

通过 powermock 的 load 将 apis 配置加载到 mock server 的内存中，但这种方式，如果一旦我们重启了 mock server，先前定义的配置就会失效。

有没有持久化的方案呢？有，就是接下来要介绍的 redis 方案。

### redis 插件

通过将 config.yaml 的 simple 配置更新为 redis，如下：

```yaml
redis:
  enable: true
  addr: 127.0.0.1:6379
  password: ""
  db: 0
  prefix: /mockserver/
```

重启 Mock 服务之后，再次 load apis 配置信息。

```bash
127.0.0.1:6379> keys *
1) "/mockserver/hello_example_gRPC"
2) "/mockserver/__REVISION__"
```

通过 redis 持久化存储的 Mock 规则的好处不言而喻。

不断补充依赖服务在正式场景下的规则，逐步完成一个与真实场景几近相同的 Mock Server，无论是开发人员效率，还是自动化测试的场景覆盖都有极大的提升。

### HTTP Mock

前面介绍的都是 gRPC 接口的 Mock，作为后端开发的我肯定更加关心的这个。如果是前端更关心的肯定是 HTTP 接口的 Mock。apis 的 Mock 也非常简单。

配置目标，无论什么情况下，都返回消息 "hello world"，在 apis.yaml 中新增如下配置：

```yaml
uniqueKey: "hello_example_http"
path: "/hello"
method: "GET"
cases:
  - response:
      simple:
        header:
          x-unit-id: "3"
          x-unit-region: "sh"
        trailer:
          x-api-vesion: "1.3.2"
        body:
          hello world!
```

通过 curl 命令测试下  mock 结果，如下：

```bash

$ curl -v http://localhost:30003/hello
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 30003 (#0)
> GET /hello HTTP/1.1
> Host: localhost:30003
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Type: application/json
< X-Unit-Id: 3
< X-Unit-Region: sh
< Date: Sun, 11 Jul 2021 08:06:57 GMT
< Content-Length: 12
< 
* Connection #0 to host localhost left intact
hello world!* Closing connection 0
```

至此，一个 HTTP 接口的 Mock 已经完成。

### 高级配置

前面的内容主要介绍如何快速上手这个新的 Mock Server。但在真实场景下，伪造数据场景复杂，只返回固定的信息肯定不满足我们的要求。

如何处理呢？这就需要 apis.yaml 支持更多的高级 Mock 规则。

#### 前置准备

还是以一个真实的场景为例吧，一个简单的用户服务的 proto 接口定义。

```proto
syntax = "proto3";

package examples.user.api;
option go_package = "github.com/poloxue/mock/examples/user/apis";

service User {
  rpc Register(GetUserRequest) returns(GetUserResponse);
  rpc SetUserDetail(SetUserDetailRequest) returns(SetUserDetailRequest);
  rpc GetUser(GetUserRequest) returns(GetUserResponse);
}

message RegisterRequest {
  string mobile = 1;
  string password = 2;
}

message RegisterResponse {
  int32 status = 1;
}

message SetUserDetailRequest {
  int64 id = 1;
  string mobile = 2;
  string username = 3;
  string email = 4;
}

message GetUserRequest {
  int64 id = 1;
  string mobile = 2;
}

message GetUserResponse {
  int64 id = 1;
  string mobile = 2;
    string username = 3;
    string email = 4;
}
```

共三个接口，分别是用户注册、设置详情以及获取用户信息。我们按 快速开始 中的步骤把 proto 的 go 文件编译，配置等准备完成。

#### 场景一 特定 ID 返回特定用户信息

如果 id 为 1000 的情况，返回用户信息如下：
```json
{
    "id": 1000,
    "mobile": "15300000001",
    "username": "poloxue",
    "email": "[poloxue123@gmail.com](mailto:poloxue123@gmail.com)"
}
```

配置 Mock 规则如下：

```yaml
uniqueKey: "get_user_gRPC"
path: "/examples.user.api.User/GetUser"
method: "POST"
cases:
  - condition:
      simple:
        items:
          - operandX: "$request.body.id"
            operator: "=="
            operandY: "1000"
    response:
      simple:
        body: |
          {"id": 1000, "mobile": "15300000001", "username": "poloxue", "email": "poloxue123@gmail.com"}
```

将配置加载到 Mock Server 中，通过 grpcurl 请求数据，将得到如下结果：

```bash
$ grpcurl -d '{"id": 1000}' -plaintext -protoset user.protoset 127.0.0.1:30002 examples.user.api.User/GetUser
{
  "id": "1000",
  "mobile": "15300000001",
  "username": "poloxue",
  "email": "poloxue123@gmail.com"
}
```

上述通过 condition 的判断，如果 $request.body.id 是 1000，则返回特定的用户数据。

#### 场景二 通过脚本返回用户数据

上述的场景按用户 ID 返回用户，如果需要返回多种场景用户，要写大量的映射规则。如果制定通用的生成规则呢？

指定一个目标，如果用户 ID 在 500 以内，返回规则为 email 为空，mobile 规则为 1530000+4 位用户 ID，用户名为 poloxue+用户ID。

新增的 condition 配置规则如下：

```bash
- condition:
      simple:
        items:
          - operandX: "$request.body.id"
            operator: "<"
            operandY: "500"
    response:
      script:
        lang: "javascript"
        content: |
          (function () {
              function pad(num, size) {
                  num = num.toString();
                  while (num.length < size) num = "0" + num;
                  return num;
              }
              return {
                code: 0,
                body: {
                    id: request.body.id,
                    username: 'username' + request.body.id,
                    mobile: '1530000' + pad(request.body.id, 4),
                },
              }
          })()
```

使用 powermock load 命令加载新的配置到 server 中。至此，如果我们使用 grpcurl 访问这个服务，得到的信息如下：

```bash
$ grpcurl -d '{"id": 100}' -plaintext -protoset user.protoset 127.0.0.1:30002 examples.user.api.User/GetUser
ERROR:
  Code: Internal
  Message: plugin(grpc): failed to unmarshal: EOF
```

因为，如果希望通过脚本生成 mock 数据，要将 powermock 替换为 powermock-v8 命令才可执行javascript 脚本。

```bash
$ grpcurl -d '{"id": 1}' -plaintext -protoset user.protoset 127.0.0.1:30002 examples.user.api.User/GetUser 
{
  "id": "1",
  "mobile": "15300000001",
  "username": "username1"
}
$ grpcurl -d '{"id": 100}' -plaintext -protoset user.protoset 127.0.0.1:30002 examples.user.api.User/GetUser
{
  "id": "100",
  "mobile": "15300000100",
  "username": "username100"
}
```
一个批量 Mock 用户数据的规则就写好了。

## 总结

整体而言， powermock 易于使用，但刚刚开源，还有些待完善的地方。但鉴于代码易读，容易扩展，与我们而言，当前的确是一个不错的选择。

## 参考资料

- [Mock Server 实践](https://tech.meituan.com/2015/10/19/mock-server-in-action.html)
- [阿里Mock平台使用方法揭秘！](https://developer.aliyun.com/article/236198)
- [powermock 代码仓库](https://github.com/bilibili-base/powermock)

博客地址：[powermock: 一个支持 gRPC 的 Mock Server](https://www.poloxue.com/posts/2021-07-17-powermock-autotest-your-code/)

