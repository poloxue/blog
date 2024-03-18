---
title: "Go 的 Http 请求系统指南"
date: 2019-09-10T16:37:17+08:00
draft: false
tags: ["Golang"]
comment: true
---


前几天在 "知乎想法" 谈到了一个话题，如何模仿学习，举了通过 net/http client 模仿 Pyhton 的requests的例子。但并未实践，难道想法真的只能是想法吗？当然不是，于是我决定先暂停一周 GO 笔记，来实践下自己的想法。

[有些新的知识，我们可以通过模仿学习](https://www.zhihu.com/pin/1112419189498933248)

本文将通过 GO 实现 requests 的 quick start 文档中的所有例子，系统学习http client的使用。虽然标题是 quick start，但其实内容挺多的。

# 快速体验

首先，我们来发起一个 GET 请求，代码非常简单。如下：

```go
func get() {
	r, err := http.Get("https://api.github.com/events")
	if err != nil {
		panic(err)
	}
	defer func() { _ = r.Body.Close() }()

	body, _ := ioutil.ReadAll(r.Body)
	fmt.Printf("%s", body)
}
```

通过 http.Get 方法，获取到了一个 Response 和一个 error ，即 r 和 err。通过 r 我们能获取响应的信息，err 可以实现错误检查。

r.Body 被读取后需要关闭，可以defer来做这件事。内容的读取可通过 ioutil.ReadAll实现。

# 请求方法

除了GET，HTTP还有其他一系列方法，包括POST、PUT、DELETE、HEAD、OPTIONS。快速体验中的GET是通过一种便捷的方式实现的，它隐藏了很多细节。这里暂时先不用它。

我们先来介绍通用的方法，以帮我们实现所有HTTP方法的请求。主要涉及两个重要的类型，Client 和 Request。

Client 即是发送 HTTP 请求的客户端，请求的执行都是由 Client 发起。它提供了一些便利的请求方法，比如我们要发起一个Get请求，可通过 client.Get(url) 实现。更通用的方式是通过 client.Do(req) 实现，req 属于 Request 类型。

Request 是用来描述请求信息的结构体，比如请求方法、地址、头部等信息，我们都可以通过它来设置。Request 的创建可以通过 http.NewRequest 实现。

接下来列举 HTTP 所有方法的实现代码。

GET
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodGet, "https://api.github.com/events", nil))
```

POST
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodPost, "http://httpbin.org/post", nil))
```

PUT
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodPut, "http://httpbin.org/put", nil))
```

DELETE
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodDelete, "http://httpbin.org/delete", nil))
```

HEAD
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodHead, "http://httpbin.org/get", nil))
```
OPTIONS
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodOptions, "http://httpbin.org/get", nil))
```

上面展示了HTTP所有方法的实现。这里还几点需要说明。

DefaultClient，它是 net/http 包提供了默认客户端，一般的请求我们无需创建新的 Client，使用默认即可。

GET、POST 和 HEAD 的请求，GO提供了更便捷的实现方式，Request 不用手动创建。

示例代码，每个 HTTP 请求方法都有两种实现。

GET
```go
r, err := http.DefaultClient.Get("http://httpbin.org/get")
r, err := http.Get("http://httpbin.org/get")
```

POST
```go
bodyJson, _ := json.Marshal(map[string]interface{}{
	"key": "value",
})
r, err := http.DefaultClient.Post(
	"http://httpbin.org/post",
	"application/json",
	strings.NewReader(string(bodyJson)),
)
r, err := http.Post(
	"http://httpbin.org/post",
	"application/json",
	strings.NewReader(string(bodyJson)),
)
```
这里顺便演示了如何向 POST 接口提交 JSON 数据的方式，主要 content-type 的设置，一般JSON接口的 content-type 为 application/json。

HEAD
```go
r, err := http.DefaultClient.Head("http://httpbin.org/get")
r, err := http.Head("http://httpbin.org/get")
```

如果看了源码，你会发现，http.Get 中调用就是 http.DefaultClient.Get，是同一个意思，只是为了方便，提供这种调用方法。Head 和 Post 也是如此。

# URL参数

通过将键/值对置于 URL 中，我们可以实现向特定地址传递数据。该键/值将跟在一个问号的后面，例如 http://httpbin.org/get?key=val。 手工构建 URL 会比较麻烦，我们可以通过 net/http 提供的方法来实现。

举个栗子，比如你想传递 key1=value1 和 key2=value2 到 http://httpbin.org/get。代码如下：

```go
req, err := http.NewRequest(http.MethodGet, "http://httpbin.org/get", nil)
if err != nil {
	panic(err)
}

params := make(url.Values)
params.Add("key1", "value1")
params.Add("key2", "value2")

req.URL.RawQuery = params.Encode()

// URL 的具体情况 http://httpbin.org/get?key1=value1&key2=value2
// fmt.Println(req.URL.String())

r, err := http.DefaultClient.Do(req)
```

url.Values 可以帮助组织 QueryString，查看源码发现 url.Values 其实是 map[string][]string。调用 Encode 方法，将组织的字符串传递给请求 req 的 RawQuery。通过 url.Values也可以设置一个数组参数，类似如下的形式：

http://httpbin.org/get?key1=value1&key2=value2&key2=value3

怎么做呢？

```go
params := make(url.Values)
params.Add("key1", "value1")
params.Add("key2", "value2")
params.Add("key2", "value3")
```

观察最后一行代码。其实，只要在 key2 上再增加一个值就可以了。

# 响应信息

执行请求成功，如何查看响应信息。要查看响应信息，可以大概了解下，响应通常哪些内容？常见的有主体内容（Body）、状态信息（Status）、响应头部（Header）、内容编码（Encoding）等。

## Body

其实，在最开始的时候已经演示Body读取的过程。响应内容的读取可通过 ioutil 实现。
```
body, err := ioutil.ReadAll(r.Body)
```
响应内容多样，如果是 json，可以直接使用 json.Unmarshal 进行解码，JSON知识不介绍了。

r.Body 实现了 io.ReadeCloser 接口，为减少资源浪费要及时释放，可以通过 defer 实现。

```go
defer func() { _ = r.Body.Close() }()
```

## StatusCode

响应信息中，除了 Body 主体内容，还有其他信息，比如 status code 和 charset 等。

```
r.StatusCode
r.Status
```

r.StatusCode 是 HTTP 返回码，Status 是返回状态描述。

## Header

响应头信息通过 Response.Header 即可获取，要说明的一点是，响应头的 Key 是不区分大小写。
```go
r.Header.Get("content-type")
r.Header.Get("Content-Type")
```
你会发现 content-type 和 Content-Type 获取的内容是完全一样的。

## Encoding

如何识别响应内容编码呢？我们需要借助 http://golang.org/x/net/html/charset 包实现。先来定义一个函数，代码如下：

```go
func determineEncoding(r *bufio.Reader) encoding.Encoding {
	bytes, err := r.Peek(1024)
	if err != nil {
		fmt.Printf("err %v", err)
		return unicode.UTF8
	}

	e, _, _ := charset.DetermineEncoding(bytes, "")

	return e
}
```

怎么调用它？

```go
bodyReader := bufio.NewReader(r.Body)
e := determineEncoding(bodyReader)
fmt.Printf("Encoding %v\n", e)

decodeReader := transform.NewReader(bodyReader, e.NewDecoder())
```

利用 bufio 生成新的 reader，然后利用 determineEncoding 检测内容编码，并通过 transform 进行编码转化。

# 图片下载

如果访问内容是一张图片，我们如何把它下载下来呢？比如如下地址的图片。

https://pic2.zhimg.com/v2-5e8b41cae579722bd6b8a612bf1660e6.jpg

其实很简单，只需要创建新的文件并把响应内容保存进去即可。

```go
f, err := os.Create("as.jpg")
if err != nil {
	panic(err)
}
defer func() { _ = f.Close() }()

_, err = io.Copy(f, r.Body)
if err != nil {
	panic(err)
}
```

r 即 Response，利用 os 创建了新的文件，然后再通过 io.Copy 将响应的内容保存进文件中。

# 定制请求头

如何为请求定制请求头呢？Request 其实已经提供了相应的方法，通过 req.Header.Add 即可完成。

举个例子，假设我们将要访问 http://httpbin.org/get，但这个地址针对 user-agent 设置了发爬策略。我们需要修改默认的 user-agent。

示例代码：

```go
req, err := http.NewRequest(http.MethodGet, "http://httpbin.org/get", nil)
if err != nil {
	panic(err)
}

req.Header.Add("user-agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0)")
```

如上便可完成任务。

# 复杂的POST请求

前面已经展示过了向 POST 接口提交 JSON 数据的方式。接下来介绍下另外几种向 POST 接口提交数据的方式，即表单提交和文件提交。

## 表单提交

表单提交是一个很常用的功能，故而在 net/http 中，除了提供标准的用法外，还给我们提供了简化的方法。

我们先来介绍个标准的实现方法。

举个例子，假设要向 http://httpbin.org/post 提交 name 为 poloxue 和 password 为 123456 的表单。

```go
payload := make(url.Values)
payload.Add("name", "poloxue")
payload.Add("password", "123456")
req, err := http.NewRequest(
	http.MethodPost,
	"http://httpbin.org/post",
	strings.NewReader(payload.Encode()),
)
if err != nil {
	panic(err)
}
req.Header.Add("Content-Type", "application/x-www-form-urlencoded")

r, err := http.DefaultClient.Do(req)
```

POST 的 payload 是形如 name=poloxue&password=123456 的字符串，故而我们可以通过 url.Values 进行组织。

提交给 NewRequest 的内容必须是实现 Reader 接口的类型，所以需要 strings.NewReader转化下。

Form 表单提交的 content-type 要是 application/x-www-form-urlencoded，也要设置下。

复杂的方式介绍完了。接着再介绍简化的方式，其实表单提交只需调用 http.PostForm 即可完成。示例代码如下：

```go
payload := make(url.Values)
payload.Add("name", "poloxue")
payload.Add("password", "123456")
r, err := http.PostForm("http://httpbin.org/post", form)
```

竟是如此的简单。

## 提交文件

文件提交应该是 HTTP 请求中较为复杂的内容了。其实说难也不难，区别于其他的请求，我们要花些精力来读取文件，组织提交POST的数据。

举个例子，假设现在我有一个图片文件，名为 as.jpg，路径在 /Users/polo 目录下。现在要将这个图片提交给 http://httpbin.org/post。

我们要先组织 POST 提交的内容，代码如下：

```go
filename := "/Users/polo/as.jpg"

f, err := os.Open(filename)
if err != nil {
	panic(err)
}
defer func() { _ = f.Close() }()

uploadBody := &bytes.Buffer{}
writer := multipart.NewWriter(uploadBody)

fWriter, err := writer.CreateFormFile("uploadFile", filename)
if err != nil {
	fmt.Printf("copy file writer %v", err)
}

_, err = io.Copy(fWriter, f)
if err != nil {
	panic(err)
}

fieldMap := map[string]string{
	"filename": filename,
}
for k, v := range fieldMap {
	_ = writer.WriteField(k, v)
}

err = writer.Close()
if err != nil {
	panic(err)
}
```

我认为，数据组织分为几步完成，如下：

- 第一步，打开将要上传的文件，使用 defer f.Close() 做好资源释放的准备；
- 第二步，创建存储上传内容的 bytes.Buffer，变量名为 uploadBody；
- 第三步，通过 multipart.NewWriter 创建 writer，用于向 buffer中写入文件提供的内容；
- 第四步，通过writer.CreateFormFile 创建上传文件并通过 io.Copy 向其中写入内容；
- 最后，通过 writer.WriteField 添加其他的附加信息，注意最后要把 writer 关闭；

至此，文件上传的数据就组织完成了。接下来，只需调用 http.Post 方法即可完成文件上传。

```go
r, err := http.Post("http://httpbin.org/post", writer.FormDataContentType(), uploadBody)
```

有一点要注意，请求的content-type需要设置，而通过 writer.FormDataContentType() 即能获得上传文件的类型。

到此，文件提交也完成了，不知道有没有非常简单的感觉。

# Cookie

主要涉及两部分内容，即读取响应的 cookie 与设置请求的 cookie。响应的 cookie 获取方式非常简单，直接调用 r.Cookies 即可。

重点来说说，如何设置请求 cookie。cookie设置有两种方式，一种设置在 Client 上，另一种是设置在 Request 上。

## Client 上设置 Cookie

直接看示例代码：

```go
cookies := make([]*http.Cookie, 0)

cookies = append(cookies, &http.Cookie{
	Name:   "name",
	Value:  "poloxue",
	Domain: "httpbin.org",
	Path:   "/cookies",
})
cookies = append(cookies, &http.Cookie{
	Name:   "id",
	Value:  "10000",
	Domain: "httpbin.org",
	Path:   "/elsewhere",
})

url, err := url.Parse("http://httpbin.org/cookies")
if err != nil {
	panic(err)
}

jar, err := cookiejar.New(nil)
if err != nil {
	panic(err)
}
jar.SetCookies(url, cookies)

client := http.Client{Jar: jar}

r, err := client.Get("http://httpbin.org/cookies")
```

代码中，我们首先创建了 http.Cookie 切片，然后向其中添加了 2 个 Cookie 数据。这里通过 cookiejar，保存了 2 个新建的 cookie。

这次我们不能再使用默认的 DefaultClient 了，而是要创建新的 Client，并将保存 cookie 信息的 cookiejar 与 client 绑定。接下里，只需要使用新创建的 Client 发起请求即可。

## 请求上设置 Cookie

请求上的 cookie 设置，通过 req.AddCookie即可实现。示例代码：

```go
req, err := http.NewRequest(http.MethodGet, "http://httpbin.org/cookies", nil)
if err != nil {
	panic(err)
}

req.AddCookie(&http.Cookie{
	Name:   "name",
	Value:  "poloxue",
	Domain: "httpbin.org",
	Path:   "/cookies",
})

r, err := http.DefaultClient.Do(req)
```

挺简单的，没什么要介绍的。

cookie 设置 Client 和 设置在 Request 上有何区别？一个最易想到的区别就是，Request 的 cookie 只是当次请求失效，而 Client 上的 cookie 是随时有效的，只要你用的是这个新创建的 Client。

# 重定向和请求历史

默认情况下，所有类型请求都会自动处理重定向。

Python 的 requests 包中 HEAD 请求是不重定向的，但测试结果显示 net/http 的 HEAD 是自动重定向的。

net/http 中的重定向控制可以通过 Client 中的一个名为 CheckRedirect 的成员控制，它是函数类型。定义如下：

```go
type Client struct {
	...
	CheckRedirect func(req *Request, via []*Request) error
	...
}
```

接下来，我们来看看怎么使用。

假设我们要实现的功能：为防止发生循环重定向，重定向次数定义不能超过 10 次，而且要记录历史 Response。

示例代码：

```go
var r *http.Response
history := make([]*http.Response, 0)

client := http.Client{
	CheckRedirect: func(req *http.Request, hrs []*http.Request) error {
		if len(hrs) >= 10 {
			return errors.New("redirect to many times")
		}

		history = append(history, req.Response)
		return nil
	},
}

r, err := client.Get("http://github.com")
```

首先创建了 http.Response 切片的变量，名称为 history。接着在 http.Client 中为 CheckRedirect 赋予一个匿名函数，用于控制重定向的行为。CheckRedirect 函数的第一个参数表示下次将要请求的 Request，第二个参数表示已经请求过的 Request。

当发生重定向时，当前的 Request 会保存上次请求的 Response，故而此处可以将 req.Response 追加到 history 变量中。

# 超时设置
Request 发出后，如果服务端迟迟没有响应，那岂不是很尴尬。那么我们就会想，能否为请求设置超时规则呢？毫无疑问，当然可以。

超时可以分为连接超时和响应读取超时，这些都可以设置。但正常情况下，并不想有那么明确的区别，那么也可以设置个总超时。

## 总超时

总的超时时间的设置是绑定在 Client 的一个名为 Timeout 的成员之上，Timeout 是 time.Duration。

假设这是超时时间为 10 秒，示例代码：

```go
client := http.Client{
	Timeout:   time.Duration(10 * time.Second),
}
```

## 连接超时

连接超时可通过 Client 中的 Transport 实现。Transport 中有个名为 Dial 的成员函数，可用设置连接超时。Transport 是 HTTP 底层的数据运输者。

假设设置连接超时时间为 2 秒，示例代码：

```
t := &http.Transport{
	Dial: func(network, addr string) (net.Conn, error) {
		timeout := time.Duration(2 * time.Second)
		return net.DialTimeout(network, addr, timeout)
	},
}
```
在 Dial 的函数中，我们通过 net.DialTimeout 进行网络连接，实现了连接超时功能。

## 读取超时

读取超时也要通过 Client 的 Transport 设置，比如设置响应的读取为 8 秒。

示例代码：
```go
t := &http.Transport{
	ResponseHeaderTimeout: time.Second * 8,
}
综合所有，Client 的创建代码如下：

t := &http.Transport{
	Dial: func(network, addr string) (net.Conn, error) {
		timeout := time.Duration(2 * time.Second)
		return net.DialTimeout(network, addr, timeout)
	},
	ResponseHeaderTimeout: time.Second * 8,
}
client := http.Client{
	Transport: t,
	Timeout:   time.Duration(10 * time.Second),
}
```

除了上面的几个超时设置，Transport 还有其他一些关于超时的设置，可以看下 Transport 的定义，还有发现三个与超时相关的定义：

```go
// IdleConnTimeout is the maximum amount of time an idle
// (keep-alive) connection will remain idle before closing
// itself.
// Zero means no limit.
IdleConnTimeout time.Duration

// ResponseHeaderTimeout, if non-zero, specifies the amount of
// time to wait for a server's response headers after fully
// writing the request (including its body, if any). This
// time does not include the time to read the response body.
ResponseHeaderTimeout time.Duration

// ExpectContinueTimeout, if non-zero, specifies the amount of
// time to wait for a server's first response headers after fully
// writing the request headers if the request has an
// "Expect: 100-continue" header. Zero means no timeout and
// causes the body to be sent immediately, without
// waiting for the server to approve.
// This time does not include the time to send the request header.
ExpectContinueTimeout time.Duration
```

分别是 IdleConnTimeout （连接空闲超时时间，keep-live 开启）、TLSHandshakeTimeout （TLS 握手时间）和 ExpectContinueTimeout（似乎已含在 ResponseHeaderTimeout 中了，看注释）。

到此，完成了超时的设置。相对于 Python requests 确实是复杂很多。

# 请求代理

代理还是挺重要的，特别对于开发爬虫的同学。那 net/http 怎么设置代理？这个工作还是要依赖 Client 的成员 Transport 实现，这个 Transport 还是挺重要的。

Transport 有个名为 Proxy 的成员，具体看看怎么使用吧。假设我们要通过设置代理来请求谷歌的主页，代理地址为 http://127.0.0.1:8087。

示例代码：

```go
proxyUrl, err := url.Parse("http://127.0.0.1:8087")
if err != nil {
	panic(err)
}
t := &http.Transport{
	Proxy:           http.ProxyURL(proxyUrl),
	TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
}
client := http.Client{
	Transport: t,
	Timeout:   time.Duration(10 * time.Second),
}

r, err := client.Get("https://google.com")
```

主要关注 http.Transport 创建的代码。两个参数，分时 Proxy 和 TLSClientConfig，分别用于设置代理和禁用 https 验证。我发现其实不设置 TLSClientConfig 也可以请求成功，具体原因没仔细研究。

# 错误处理

错误处理其实都不用怎么介绍，GO中的一般错误主要是检查返回的error，HTTP 请求也是如此，它会视情况返回相应错误信息，比如超时、网络连接失败等。

示例代码中的错误都是通过 panic 抛出去的，真实的项目肯定不是这样的，我们需要记录相关日志，时刻做好错误恢复工作。

# 总结

本文以 Python 的 requests 文档为指导方向，整理了 requests 快速入门文档中的案例在 GO 的是如何实现的。要说明的是， GO 其实也提供了对应于 requests 的克隆版本，[github地址](https://github.com/levigross/grequests)。暂时我也还没有看，有兴趣的朋友可以去研究一下。

博文地址：[Go 的 Http 请求系统指南](https://www.poloxue.com/posts/2019-09-10-the-guide-for-go-http-client/)
