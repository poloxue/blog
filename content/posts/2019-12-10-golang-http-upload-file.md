---
title: "Go 如何实现 HTTP 文件上传"
date: 2019-12-10T15:25:18+08:00
draft: false
tags: ["Golang"]
comment: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-24-http-upload-file-in-golang-01.png)

早前写过一篇文章，[Go HTTP 请求 QuickStart](https://www.poloxue.com/posts/2019-09-10-the-guide-for-go-http-client/)。当时，主要参考 Python 的 requests 大纲介绍 Go 的 net/http 如何发起 HTTP 请求。

最近，尝试录成它的视频，[访问地址](https://www.bilibili.com/video/av77753893 "访问地址")。发现当时写得挺详细的，发现当时虽然写得比较详细，但也只是介绍用法，可能不知其所以然。比如文件上传那部分，如果不了解 http 文件上传协议 [RFC 1867](https://tools.ietf.org/html/rfc1867 "RFC 1867")，就很难搞懂为什么代码这么写。

今天，就以这个话题为基础，介绍下 Go 如何实现文件上传。

相关代码请访问 [httpdemo/post](https://github.com/poloxue/go-series-video/tree/master/httpdemo/post)。本文视频地址：[Go 上传文件](https://www.bilibili.com/video/av78747321/)

# 简介

简单来说，HTTP 上传文件可以分三个步骤，分别是组织请求体，设置 Content-Type 和发送 Post 请求。POST 请求就不用介绍了，主要关注请求体和请求体内容类型。


请求体，即 request body，常用于 POST 请求上。请求体并非 POST 特有，GET 也支持，只不过约定俗成的规定，服务端一般会忽略 GET 的请求体。

Content-Type 是什么？

因为，请求体的格式并不固定，可能性很多，为了明确请求体内容类型，HTTP 定义了一个请求头 Content-Type。

常见的 Content-Type 选项有 `application/x-www-form-urlencoded`（默认的表单提交）、`application/json`（json）、`text/xml`（xml 格式）、`text/plain`（纯文本）、`application/octet-stream`（二进制流）等。

# 提交表单

文件上传可以理解为是提交表单的特例，先通过表单提交这个简单的例子介绍下整个流程。

如下是表单提交的 HTTP 请求文本。

```http
POST http://httpbin.org/post HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=poloxue&password=123456
```

Content-Type 是 `application/x-www-form-urlencoded`，数据通过 urlencoded 方式组织。

先用 html 的 form 表单实现。如下：

```html
<form method="post">
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="submit">
</form>
```

通过 Post 提交 form 表单，Content-Type 默认是 `application/x-www-form-urlencoded`。

Go 的实现代码：

```go
data := make(url.Values)
data.Set("username", "poloxue")
data.Set("password", "123456")

// 按 urlencoded 组织数据
body, _ := data.Encode()

// 创建请求并设置内容类型
request, _ := http.NewRequest(
    http.MethodPost,
    "http://httpbin.org/post",
    bytes.NewReader(body),
)

request.Header.Set(
    "content-type",
    "application/x-www-form-urlencoded",
)

http.DefaultClient.Do(request)
```

回想下前面说的三个步骤，组织请求体数据、设置 Content-Type 和发送请求。

Go 的 net/htp 包还提供了一个更简洁的写法，http.Post。

```go
http.Post(
    "http://httpbin.org/post",
    "application/x-www-form-urlencoded",
    bytes.NewReader(body),
)
```

# 上传文件 RFC 1867

文件上传的需求很常见，但默认的 form 表单提交方式并不支持。

如果是单文件上传，通过 body 二进制流就可以实现。但如果是一些更复杂的场景，如上传多文件，则需要自定义上传协议，而且客户端和服务端都要提供相应的支持。

文件上传这种常见需求，如果有一套标准岂不更好。为了解决这个问题，[RFC 1867](https://tools.ietf.org/html/rfc1867 "RFC 1867") 就诞生了，它主要内容有：

- input 标签的类型增加一个 file 选项；
- form 表单的 enctype 增加 `multipart/form-data` 选项；

如下是一个支持文件提交的 form 表单。

```html
<form
    action="http://httpbin.org/post"
    method="post"
    enctype="multipart/form-data"
>
  <input type="text" name="words"/>
  <input type="file" name="uploadfile1">
  <input type="file" name="uploadfile2">
  <input type="submit">
</form>
```

提交表单后，将会看到请求的内容大致形式，如下：

```HTTP
POST http://httpbin.org/post HTTP/1.1
Content-Type: multipart/form-data; boundary=285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350

multipart/form-data; boundary=285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
Content-Disposition: form-data; name="uploadFile1"; filename="uploadfile1.txt"
Content-Type: application/octet-stream

upload file1
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
Content-Disposition: form-data; name="uploadFile1"; filename="uploadfile2.txt"
Content-Type: application/octet-stream

upload file2
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
Content-Disposition: form-data; name="words"

123
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350--
```

注：如果使用 chrome 浏览器的开发者工具，为了性能考虑，无法看到看到这部分内容。而且，如果提交的是二进制流，只是一串乱码，也没什么可看的。

Content-Type 除了 `multipart/form-data`，还另外多了 `boundary=xxx` 的内容。`boundary`是边界的意思，相当于 `application/x-www-form-urlencoded` 方式中的 `&`，用于分隔不同 input 字段。`boundary` 之所以这么复杂，因为，一般的文本内容使用了 `&` 就能分离，但如果是文件流，`&` 可能和内容冲突，对边界的唯一性要求更高。

`multipart/form-data` 内容的详细格式就不介绍了。继续说如何用 Go 实现这个功能。

# Go 实现代码

如何使用 Go 实现文件上传？

主体逻辑依然是组织数据、设置 Content-Type 和发送请求这三步。但这部分数据的组织比 form 表单的 urlencoded 的方式要复杂的多。

Go 的简洁性这时就体现出来了，因为，标准库 `mime/multipart` 已经提供了非常好用的方法，无需自己手动组织。

假设，现在要实现前面 form 表单的功能，即提交两个文件，uploadfile1、uploadfile2，和一个字段 words。

首先，创建一个用于保存数据的 `byte.Buffer` 类型的变量，`body`，在它之上创建一个 `multipart.Writer`，用这个 `writer` 组织将要提交的数据。代码如下：

```go
bodyBuf := &bytes.Buffer{}
writer := multipart.NewWriter(payloadBuf)
```

先组织文件内容，两个文件的组织逻辑相同，就以 uploadfile1 为例进行介绍。在 `writer` 之上创建一个 `fileWriter`，用于写入文件 uploadFile1 的内容，

```go
fileWriter, err := writer.CreateFormFile("uploadFile1", filename)
```

打开要上传的文件，uploadfile1，将文件内容拷贝到 `fileWriter`中，如下：

```go
f, err := os.Open("uploadfile1")
    ...
io.Copy(fileWriter, f)
```

添加字段就非常简单了，假设设置 `words` 为 123，代码如下：

```go
writer.WriteField("words", "123")
```

完成所有内容设置后，一定要记得关闭 `Writer`，否则，请求体会缺少结束边界。

```go
writer.Close()
```

完成了数据的组织。

接下来，只要将数据设置到 `http.Post` 就好了。

```go
r, err := http.Post(
    "http://httpbin.org/post",
    writer.FormDataContentType(),
    body,
)
```

完成了支持文件上传的表单提交。

# 总结

本篇文章主要介绍了如何使用 Go 实现文件上传，本质上是组织提交文件的请求体。而为了能清晰地了解请求体的组织过程，就必须清楚相关的 HTTP 协议，[rfc 1867](https://tools.ietf.org/html/rfc1867 "rfc 1867")。

博文地址：[Go 如何实现 HTTP 文件上传](http://localhost:1313/posts/2019-12-10-golang-http-upload-file/)
