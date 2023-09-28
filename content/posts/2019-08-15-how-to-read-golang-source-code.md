---
title: "如何阅读 Go 源码"
date: 2019-08-15T17:40:14+08:00
draft: false
comment: true
tags: ["Golang"]
---

Go 的源码在安装包的 src/ 目录下。怎么看它的源码呢？直接看吧！没人教的情况下，只能自己撸了。当然，这种内容一般也不会有人教。

怎么撸？

Go 源码中，应该可分为与语言息息相关的部分，和官方提供的标准库。与语言实现相关的肯定是最难的，不是那么容易理解。可以先主要看标准库，其他的可以先大概了解下。

先把源码目录整体扫一遍，大概看看涉及了哪些模块，然后再挑自己喜欢的部分进行更深一步的学习与研究。建议每个目录都简单写个 hello world，如此的体悟会更深。如果连 hello world 也写不出来，这个模块的源码暂时就没必要研究了，先学好基础吧。毕竟，包的使用不仅与语言相关，还涉及具体场景和实现原理，这都是要学习的。

对包的使用熟悉理解后，就可以阅读源码了，但此时最好还是不要太抠细节，求理解涉及设计思想，整体流程。源码阅读可以通过画 UML 的方式辅助，从纵向和横向帮助理解。代码设计时，一般最容易想到的就是按顺序方式写，很快就能搞定。但当项目变大，抽象的模块会越来越多，抽象出接口和具体的实现，实现可能包含其他类型的组合。搞明白这些关系，对于理解源码实现会较有帮助。

如果能顺利经过前面两步，接下来的源码阅读就比较简单了。而且 Go 语言的特点就是简洁易读，没什么语法糖。当然，如果是一些实现比较复杂的包，你还需知道它们的底层原理，就比如 net/http 包，你得对 http 协议熟悉到一定程度，才可能从细节研究源码实现。 

可能是我闲的蛋疼，准备试着先从第一步出发，整体撸一下 Go 的源码中包含的模块，没事的时候就更新一点进去。等把这些大致撸完一遍，感觉我的 [Golang 之旅](https://zhuanlan.zhihu.com/poloxue-go) 专栏又可以多出很多写作素材了。

我的环境是 Go 1.11。关于每个模块，我会把读过的一些文章放在下面，由于只是粗略阅读，并不能保证读过的每篇文章都是精品。

补充：

2019年8月8日 凌晨 01:13， 大概花了两个多星期的零碎时间，简单撸完了一版。总的感觉，还是有很多地方理解不够，希望后面可以按前面说的思路，按包逐步进行源码解剖。

---

## archive

包含了文件归档的相关内容，其中涉及了两个包，分别是 tar 和 zip。

archive/tar，即归档，如果了解 Linux 下的 tar 命令，可与之对应理解。如果要在归档基础上进行压缩，还要借助 compress 下的相关包。提醒一点，是使用时要注意理解归档与压缩的区别。

相关阅读：

[鸟哥的文件与文件系统的压缩与打包](http://cn.linux.vbird.org/linux_basic/0240tarcompress.php)  
[archive/tar 实现打包压缩及解压](https://learnku.com/articles/23433/golang-learning-notes-four-archivetar-package-compression-and-decompression)

archive/zip，与 zip 格式压缩文件操作相关的包，使用方法与 tar 很类似。在寻找与 zip 包相关的资料时，了解到 zip 的作者年仅 37 岁就逝世了，而全世界所有使用 zip 压缩的文件开头部分都有他的名字 "PK"，而我们识别一个文件是否是 zip 正是通过这种方法。

相关阅读：

[archive/zip 实现压缩与解压](https://learnku.com/articles/23434/golang-learning-notes-five-archivezip-to-achieve-compression-and-decompression)  
[zip 的百度百科](https://baike.baidu.com/item/zip/16684862)

## bufio

实现了缓冲 IO 的功能，通过包裹 io.Reader 或 io.Writer 函数创建新的 Reader 或 Writer 实例，并且这些新创建的实例提供了缓冲的能力。使用方法非常简单，达到指定缓冲大小，触发写或读操作，如未达到要求，可用 Flush 方法刷新。

相关阅读：

[Introduction to bufio package in Golang](https://medium.com/golangspec/introduction-to-bufio-package-in-golang-ad7d1877f762)  
[In-depth introduction to bufio.Scanner in Golang](https://medium.com/golangspec/in-depth-introduction-to-bufio-scanner-in-golang-55483bb689b4)  
[bufio - 缓存 IO](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter01/01.4.html)  

## builtin

Go 语言中的内置类型、函数、变量、常量的声明。暂时看来，没啥可深入阅读的，应该结合 Go 的内部实现进行阅读。

## bytes

主要是关于 byte slice 操作的一些函数。由于 []byte 也可用于表示 string，故其中的函数、方法与 strings 很类似，比如 Join、Split、Trim、 Contains 等。

相关阅读：

[Go 语言学习 - bytes 包](https://www.jianshu.com/p/bc7baa8bb286)  
[Go Walkthrough: bytes+ + strings](https://medium.com/go-walkthrough/go-walkthrough-bytes-strings-packages-499be9f4b5bd)

## cmd

Go 命令工具集的实现代码，如 go、gofmt、godoc、pprof 等，应该主要是和 Go 语言实现相关性较大，比较底层。每个命令都够研究一段时间了，特别是 go 命令，并且前提是你的计算机底层原理的功底要足够优秀。

网上搜索下，关于它的资料比较少。

相关阅读：

[Go 官网之 Command go](https://golang.org/pkg/cmd/go/)

## compress

之前提到 archive 包中是归档相关操作，而相对的 compress 包主要与压缩相关。主要实现了几种主流的压缩格式，如 bzip2、flate、gzip、lzw、zlib。

compress/bzip2，常见的 .bz2 结尾的压缩文件格式基本可用这个包操作，要与 tar 结合使用。

compress/gzip，常见的 .gz 结尾的压缩文件格式基本可用这个包操作，要与 tar 结合使用。

compress/flate，flate 应该主要是 zip 用的压缩算法，如果阅读了前面的 archive/zip 的源码，就会发现其中导了这个包。

compress/zlib， compress/lzw 基本与上面同理，应该都是某种压缩算法实现。因为我对压缩算法没什么太深的研究，暂时了解个大概就好了，希望没有介绍错误。

相关阅读：

[Go 官网之 compress](https://golang.org/pkg/compress/)

## container

我们知道，Go 内置的数据结构很少，只有数组、切片和映射。除此以外，其实还有部分的结构放在了 container 包中，heap 堆、list 双端队列，ring 回环队列。

它们的使用非常简单，基本就是增删改查。

相关阅读：

[container 容器数据类型：heap、list 和 ring](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter03/03.3.html)

## context

读这个包之前，得首先熟悉 Go 的并发代码如何编写，了解 Done channel 如何实现向所有 goroutine 发送广播信号。Go 的并发单元称为 goroutine，但是不同 goroutine 之间并没有父子兄弟关系，为了更好地并发控制，context 包就诞生了。它可以实现在不同 goroutine 间安全地传递数据以及超时管理等。

相关阅读：

[Go 译文之通过 context 实现并发控制](https://zhuanlan.zhihu.com/p/72916991)  
[深度解密 Go 语言之 Context](https://zhuanlan.zhihu.com/p/68792989)  

## crypto

加密相关，涉及内容有点多，包含了各种常用的加密算法实现，比如对称加密啊 AES、DES 等，公私钥加密 rsa、dsa 等，散列算法 sha1、sha256 等，随机数 rand 也有，不知道和 math 的随机有什么区别。没有找到一篇综合性介绍的文章，毕竟比较复杂了，如果要看它们的源码，得先要大概了解下每个加密算法的原理，才好逐一突破。

相关阅读：

[Go 官网之 crypto](https://golang.org/pkg/crypto/)

## database

封装了一套用于数据库操作的通用接口，实现了数据库连接管理，支持连接池功能。真正使用时，我们需要引入相应的驱动，才能实现指定类型数据库的操作。

一个简单的例子。
```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "username:password@tcp(127.0.0.1:3306)/test")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
}
```

github.com/go-sql-driver/mysql 便是提供的 MySQL 驱动。具体的查询执行都是通过调用驱动实现的 db 接口中的方法。

相关阅读：

[database/sql-SQL/SQL-Like 数据库操作接口](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter07/07.1.html)  
[关于Golang中database/sql包的学习笔记](https://segmentfault.com/a/1190000003036452)

## debug

和调试相关，具体内容比较复杂，我也不是很懂。内部有几个包，如 dwarf、elf、gosym、macho、pe、plan9obj。

dwarf，可用于访问可执行文件中的 DWARF 信息。具体什么是 DWARF 信息呢？官网有个 PDF，具体介绍了什么是 DWARF，有兴趣可以看看。它主要是为 UNIX 下的调试器提供必要的调试信息，例如 PC 地址对应的文件名行号等信息，以方便源码级调试。

相关阅读：

[dwarf-2.0.0 调试信息格式](http://dwarfstd.org/doc/dwarf-2.0.0.pdf)
[DWARF, 说不定你也需要它哦](https://www.jianshu.com/p/20dfe4fe1b3f)

elf，用于访问 elf 类型文件。elf，即可执行与可连接格式，常被称为 ELF 格式，有三种类型：

- 可重定位的对象文件（Relocatable file），由汇编器汇编生成的 .o 文件
- 可执行性的对象文件（Executable file），可执行应用程序
- 可被共享的对象文件（Shared object file），动态库文件，也即 .so 文件

相关阅读：

[readelf elf文件格式分析](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/readelf.html)
[The 101 of ELF files on Linux: Understanding and Analysis](https://linux-audit.com/elf-binaries-on-linux-understanding-and-analysis/)

gosym，用于访问 Go 编译器生成的二进制文件之中的 Go 符号和行信息等，暂时还没怎么看。在 medium 发现个系列文章，介绍了 Go 中 debug 调试器的实现原理，相关阅读部分是系列的第二篇文章。

相关阅读：

[Making debugger in Golang (part II)](https://medium.com/golangspec/making-debugger-in-golang-part-ii-d2b8eb2f19e0)

macho，用于访问 Mach-O object 格式文件。要阅读这段源码，同样需要先了解什么是 Mach-O，它是 Mach object 文件格式的缩写，用于可执行文件、目标代码、内核转储的文件格式。

相关阅读：

[维基百科-Mach-O](https://zh.wikipedia.org/wiki/Mach-O)  
[Go package - debug/macho](https://golang.org/pkg/debug/macho/)

pe，实现了访问 PE 格式文件，PE 是 Windows 系统可移植的可执行文件格式。

相关阅读：

[WIKI - Portable Executable](https://en.wikipedia.org/wiki/Portable_Executable)  
[Go package - debug/pe](https://golang.org/pkg/debug/pe/)

plan9obj，用于访问 plan9 object 格式文件。

暂未找到关于 plan9object 的介绍文章。我们主要学习的话，主要应该是集中在 elf 和 gosym 两个格式。

相关阅读：

[Go package - debug/plan9obj](https://golang.org/pkg/debug/plan9obj/)

## encoding

主要关于我们常用到的各种数据格式的转化操作，或也可称为编解码，比如 JSON、XML、CSV、BASE64 等，主要的模块有：

encoding/json，json 处理相关的模板，通用方式，我们可以将解析结果放到 map[string]interface{} 解析，也可以创建通用结构体，按 struct 方式进行。

encoding/xml，基本和 encoding/json 类似。但因为 XML 比 json 要复杂很多，还涉及一些高级用法，比如与元素属性相关等操作。

encoding/csv，csv 数据格式解析。

encoding/binary，可用于处理最底层的二进制数据流，按大小端实现 []byte 和整型数据之间的转化。

其他诸如 hex、gob、base64、base32、gob、pem、ascii84 等数据格式的操作都是类似的，有兴趣可以都尝试一下。

相关阅读：

[Golang 下的 encoding 相关模块的使用](https://www.jianshu.com/p/772ca3c6c7ed)
[Go 标准库文档翻译之 encoding/xml](http://blog.huangz.me/2017/go-stdlib-encoding-xml.html)
[Golang 中 byte 转 int 涉及到大小端问题吗](https://www.zhihu.com/question/327537211)
[使用 Go 语言标准库对 CSV 文件进行读写](https://xiequan.info/%E4%BD%BF%E7%94%A8go%E8%AF%AD%E8%A8%80%E6%A0%87%E5%87%86%E5%BA%93%E5%AF%B9csv%E6%96%87%E4%BB%B6%E8%BF%9B%E8%A1%8C%E8%AF%BB%E5%86%99/)
[Go Walkthrough: encoding package](https://medium.com/go-walkthrough/go-walkthrough-encoding-package-bc5e912232d)

## errors

Go 的错误处理主要代码就是它。很遗憾的是，打开源码后发现，就几行代码哦。主要是因为 Go 的错误类型只是一个接口而已，它的源码非常简单。

```go
package errors

// New returns an error that formats as the given text.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

Go 默认只提供了最简单的实现，就上面这几行代码。真的是 awesome、amazing，哈哈。但正是因为简单，扩展出自己的 error 变得很简单。比如，有些开发者认为 Go 的错误处理太简单，开发了一些包含 call stack trace 的 error 包。

相关阅读：

[github.com/pkg/errors](https://github.com/pkg/errors)  
[error-handling-and-go](https://blog.golang.org/error-handling-and-go)  
[What I Don’t Like About Error Handling in Go](https://opencredo.com/blogs/why-i-dont-like-error-handling-in-go/)  

## expvar

主要是用于 Go 程序运行时的指标记录，如 HTTP 服务在加入 expvar 后，我们可以通过 /debug/vars 返回这些指标，返回的数据是 JSON 格式的。

它的源码不多，也就大约 300 行代码，重点在它使用方法。

相关阅读：

[标准库 EXPVAR 实战](http://blog.studygolang.com/2017/06/expvar-in-action/)
[Monitoring apps with expvars and Go](https://medium.com/@piotrrojek/monitoring-apps-with-expvar-and-go-6d314267ee9f)

## flag

用于命令行参数解析的包，比如类似命令参数 grep -v grep，具体操作的时候要获取 -v 后的参数值。很常用的功能，如果纯粹自己实现是比较繁琐的。

相关阅读：

[flag-命令行参数解析](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter13/13.1.html)

## fmt

从包名就可以知道，fmt 主要和格式化相关，关于什么的格式化呢？主要是字符串的格式化，它的用法和 C 中 printf 都很类似。当然，除了实现 C 的用法，还提供了一些 Go 特有的实现。

相关阅读：

[Go Walkthrough: fmt](https://medium.com/go-walkthrough/go-walkthrough-fmt-55a14bbbfc53)

## go

似乎是核心工具使用的包。

## hash

hash 包主要定义了不同的 hash 算法的统一接口。而具体的 hash 算法实现有的直接 hash 的下层，比如 crc32、crc64，即 32 位循环冗余校验算法和 64 位循环冗余校验算法。而 md5 hash 算法在 crypto/md5 下，同样实现了 hash 的相关接口。

相关阅读：

[常见哈希函数 FNV 和 MD5](https://studygolang.com/articles/1121)

## html

Go 标准库里的 html 包功能非常简单，大概了看下，主要是关于 html 文本的处理，例如该如何对 html 代码做转义。如果想支持 html 的解析，go 官方 [github](https://github.com/golang) 下还提供了一个 [net](https://github.com/golang/net) 仓库，其中有个 [html](https://github.com/golang/net/tree/master/html) 的工具包。而 goquery 也是基于它实现的。

标准库的 html 目录下还有 template，html 的模板渲染工具，通过与 net/http 相结合，再加上一个数据库 orm 包，简单的 web 开发就可以开始了。

相关阅读：

[Easy way to render HTML in Go](https://medium.com/@thedevsaddam/easy-way-to-render-html-in-go-34575f858026)  
[Go 语言解析 html](https://studygolang.com/articles/4602)  

## image

Go 2D 图像处理库，支持创建 2D 处理的方法函数，图片创建、像素、颜色设置，然后进行绘制。主要支持 png、jpeg、gif 图片格式。

相关阅读：

[golang中image包用法](https://www.cnblogs.com/msnsj/p/4242572.html)  
[golang 中 image/draw 包用法](https://studygolang.com/articles/3396)  
[Golang 绘图技术](https://www.cnblogs.com/ghj1976/p/3443638.html)  

## index

目录为 index，其中只有一个包 index/suffixarray，称为后缀数组。具体算法没仔细研究，大致是将子字符串查询的时间复杂度降低到了 $log_n$。

使用非常简单，官网已经提供了一个例子。

```go
// create index for some data
index := suffixarray.New(data)

// lookup byte slice s
offsets1 := index.Lookup(s, -1) // the list of all indices where s occurs in data
offsets2 := index.Lookup(s, 3)  // the list of at most 3 indices where s occurs in data
```

相关阅读：

[Go package - index/suffixarray](https://golang.org/pkg/index/suffixarray/)
[suffix array 后缀数组算法心得](https://www.cnblogs.com/zzhzz/p/7532937.html)

## internal

内部实现，比较复杂。

## io

Go 的标准库中，为 io 原语提供了基本的接口和实现，帮助字节流的读取。接口主要就是 io.Reader 和 io.Writer。io 包提供了一些常用资源的接口实现，比如内存、文件和网络连接等资源进行操作。

阅读 io 包的源码，会发现很多接口都是基于具体的能力定义，最简单的有 Reader（读）、Writer（写）、Closer（关闭）、Seeker（偏移），一个接口一个方法，非常灵活。组合的接口还有 ReaderWriter（读写）、ReadeCloser（读与关）、WriteCloser（读写关） 和 ReadWriteCloser（读写关）等。整体理解，我们将会对 Go 接口是基于是鸭子模型的说法更有体会，

相关阅读：

[Go 中 io 包的使用方法](https://segmentfault.com/a/1190000015591319)  
[基本的 IO 接口](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter01/01.1.html)  
[Streaming IO in Go](https://medium.com/learning-the-go-programming-language/streaming-io-in-go-d9350793118)  

## log

Go 的日志包，通过记录日志可以方便我们进行问题调试。log 包的核心源码并不多，总共也就三百多行，其中注释就占了差不多一百行。主要是因为它提供的功能很少，只有基础的日志格式化，还有 Print、Panic、Fatal 三种日志打印函数。连错误级别没提供。如果要使用的话，还需要借助一些第三方的包。相关阅读中提供了一个 "Go 日志库集合" 的文章，具体我也没有深入研究。

相关阅读：

[Go log 日志](https://www.flysnow.org/2017/05/06/go-in-action-go-log.html)  
[Go 日志库集合](https://www.ctolib.com/topics-123640.html)  

## math

主要是关于数学计算方面的函数，一些数学常量，比如 PI（圆周率）、E（自然对数）等，就在其中，还有如四舍五入方面的函数 Round、Floor、Ceil、最大值 Max、最小值 Min，复杂的数学运算，比如幂运算、对数、三角函数肯定也有的，其他诸如随机数之类的函数也在其中。打开 math 源码文件夹，发现里面有大量的汇编代码，数学相对片底层，对性能要求会比较高，有必要用汇编实现。

math 包，直接看官方文档就好了，一般看了就可以用，没什么业务场景、具体原理需要了解，毕竟大家都学过数学。如果要看汇编实现，那就复杂了。有兴趣可以研究一下。

相关阅读：

[Go 官网 math](https://golang.org/pkg/math/)

## mime

要了解 mime 包的使用，得先了解什么是 MIME，全称 Multipurpose Internet Mail Extension，即多用途互联网邮箱扩展类型。最初设计的目标是为了在发送邮件时，附加多媒体内容。后来，MIME 在 HTML 中也得到了支持。

其中主要有四个函数，AddExtensionType、TypeByExtension、FormatMediaType 和 ParseMediaType。前后两组函数似乎都是针对 MediaType 的互操作。

相关阅读：

[Go 标准库学习 mime](https://www.cnblogs.com/wanghui-garcia/p/10401375.html)  
[Go package - mime](https://golang.org/pkg/mime/)

## net

网络相关，涉及内容比较多，有种吃不消的感觉。

底层的实现 socket 就在 net 包下，主要是一些底层协议的实现，比如无连接的 ip、udp、unix(DGRAM)，和有连接的 tcp、unix(STREAM) 都可以在 net 包找到。

应用层协议，http 协议实现在 net/http 包含客户端服务端，rpc 在 net/rpc，邮件相关的 net/mail、net/smtp 等。net/url 是与 url 处理相关的函数，比如 url 字符串解析，编码等。

相关阅读：

[golang net 包学习笔记](https://studygolang.com/articles/10165)
[Go 官方库 RPC 开发指南](https://colobu.com/2016/09/18/go-net-rpc-guide/)
[Go 爬虫必备之 HTTP 请求 QuickStart](https://zhuanlan.zhihu.com/p/61355955)
[Sending HTML emails using templates in Golang](https://medium.com/@dhanushgopinath/sending-html-emails-using-templates-in-golang-9e953ca32f3d)

## os

os 包主要实现与操作系统相关的函数，并且是与平台无关的。它的设计是 UNIX 风格的，并且采用 Go 错误处理风格。发生错误将返回的 error 类型变量。比如 Open、Stat 等操作相关的函数。

os 包的目标是统一不同操作系统的函数。如果大家读过那本 UNIX 环境高级编程，你会发现 os 包中的函数与 Unix 的系统调用函数都很相似。

除了 os 包，该目录下还有几个包，分别是 os/exec、os/signal 和 os/user，如下：

os/exec，帮助我们实现了方便执行外部命令的能力。

os/signal，Unix-Like 的系统信号处理相关函数，Linux 支持 64 中系统信号。

os/user，与系统用户相关的库，可用于获取登录用户、所在组等信息。

相关阅读：

[[译]使用 os/exec 执行命令](https://colobu.com/2017/06/19/advanced-command-execution-in-Go-with-os-exec/)
[Go package - os](https://golang.org/pkg/os/)

## path

path 包实现了路径处理（通过 / 分隔）相关的一些常用函数，常用于如文件路径、url 的 path。不适合 Windows 的 \ 和磁盘路径处理。

主要包含的函数有 Base、Clean、Dir、Ext、IsAbs、Join 等函数。如 Base 可用于获取路径的最后一个元素，Dir 获取路径目录，Ext 获取文件扩展、IsAbs 判断是否为绝对路径，Join 进行路径连接等。

相关阅读：

[Go package - path](https://golang.org/pkg/path/)

## plugin

plugin 包是 Go 1.8 出现的包，为 Go 增加了动态库加载的能力，当前只支持 Linux 和 MacOS。但这个包的应用并不是很方便，生成和使用库文件的环境有一定的要求。

相关阅读：

[如何评价 Go 标准库中新增的 plugin 包](https://www.zhihu.com/question/51650593/answer/747619907)  
[writing-modular-go-programs-with-plugins](https://link.zhihu.com/?target=https%3A//medium.com/learning-the-go-programming-language/writing-modular-go-programs-with-plugins-ec46381ee1a9)  
[calling-go-functions-from-other-languages](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/go-cshared-examples)  
[gosh-a-pluggable-command-shell-in-go](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/go-plugin-example)  

## reflect

与反射相关的函数函数，通过反射可以实现运行时动态创建、修改变量，进行函数方法的调用等操作，获得本属于解释语言的动态特性。要阅读反射包源码，重点在理解变量的两个组成，即类型和值，反射的核心操作基本都是围绕它们进行。reflect.ValueOf 与 reflect.TypeOf 是我们常用的两个方法。

相关阅读：

[Go 译文之如何使用反射](https://zhuanlan.zhihu.com/p/68241407)
[Go 译文之如何使用反射（二）](https://zhuanlan.zhihu.com/p/69752893)
[Go package - reflect](https://golang.org/pkg/reflect/)

## regexp

Go 的正则包，用于正则处理。基本是每种语言都会提供。其中涉及的方法大致可分为几个大类，分别是 Compile 编译、Match 匹配、Find 搜索、Replace 替换。

正则的源码实现还真是不想看。感觉正则都没有完全理清楚，扯源码有点坑。特别头大。

相关阅读：

[Go 官网之 regexp](https://golang.org/pkg/regexp/)
[Golang-Regex-Tutorial](https://github.com/StefanSchroeder/Golang-Regex-Tutorial)

## runtime

runtime 是与 Go 运行时相关的实现，我们可以通过它提供的一些函数控制 goroutine。关于 Go 进程的启动流程、GC、goroutine 调度器等，也是在 runtime 中实现，同样需要我们好好阅读 runtime 代码了解。除此以为，cgo、builtin 包的实现也是在 runtime。

相关阅读：

[说说 Golang 的runtime](https://zhuanlan.zhihu.com/p/27328476)
[golang internals](https://www.cnblogs.com/genius0101/archive/2012/04/16/2447147.html)
[Go package - runtime](https://golang.org/pkg/runtime/)

## sort

定义了排序的接口，一旦某个类型实现了排序的接口，就可以利用 sort 中的函数实现排序。通过阅读源码，我发现默认支持排序的类型包括 int、float64、string。sort 中还有个 search 文件，其中主要是已排序内容二分查找的实现。

我们都知道，排序算法很多，比如插入排序、堆排序与快速排序等，sort 包都已经实现了，并且不用我们决定使用哪种算法，而是会依据具体的数据决定使用什么算法，并且一次排序不一定只要了一种算法，而可能是多种算法的组合。如何做算法选择可以通过阅读 sort.go 文件中的 quickSort 函数了解。

相关阅读：

[Sort - 排序算法](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter03/03.1.html)
[The 3 ways to sort in Go](https://yourbasic.org/golang/how-to-sort-in-go/)

## strconv

关于字符串与其他类型转化的包，名字全称应该是 string convert，即字符串转化。比如整型与字符串转化的 Itoa 与 Atoi，浮点型与字符串的转化 ParseFloat 与 FormatFloat，布尔型与字符串转化 ParseBool 与 FormatBool 等等。

相关阅读：

[Golang 中 strconv 的用法](https://studygolang.com/articles/5003)
[官方文档 - strconv](https://golang.org/pkg/strconv/)

## strings

针对字符串的操作函数，前面也提过到，因为 []byte 也可用于表示字符串，strings 中的很多函数在 bytes 包也有类似的实现，比如 Join、Split、Trim，大小写转化之类的函数等。

相关阅读：

[strings - 字符串操作](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter13/13.1.html)

## sync

Go 推荐以通信方式（channel）实现并发同步控制，但传统机制也是支持的，比如锁机制、条件变量、WaitGroup、原子操作等，而它们都是由 sync 提供的。其中，原子操作在 sync/atomic 包下。

除此之外，sync 中还有个临时对象池，可以实现对象复用，并且它是可伸缩且并发安全的。

相关阅读：

[浅谈 Golang sync 包的相关使用方法](https://deepzz.com/post/golang-sync-package-usage.html)

## syscall

系统调用，从名字就能知道，这个包很复杂。系统调用是实现应用层和操作底层的接口，不同系统之间的操作常常会有一定的差异，特别是类 Unix 与 Windows 系统之间的差异较大。

如果想要寻找 syscall 的使用案例，我们可以看看 net、os、time 这些包的源码。

如果要看这部分源码，当前的想法是，我们可以只看 Linux 的实现，架构的话，如果想看汇编，可以只看 x86 架构。

暂时研究不多，不敢妄言。

相关阅读：

[视频笔记：Go 和 syscall](https://blog.lab99.org/post/golang-2017-10-05-video-guide-to-syscall.html)  
[Go package - syscall]()

## testing

Go 中测试相关的实现，比如单元测试、基准测试等。Go 推荐的测试方式采用表格驱动的测试方式，即非每种情况都要写一个单独的用例，而是通过列举输入、期望输出，然后执行功能并比较期望输出与实际输出是否相同。

一个简单的测试用例。

```go
func TestSum(t *testing.T) {
	var sumTests = []struct {
		a        int
		b        int
		expected int
	}{
		{1, 1, 2},
		{2, 1, 3},
		{3, 2, 5},
		{4, 3, 7},
		{5, 5, 10},
		{6, 8, 14},
		{7, 13, 20},
	}

	for _, tt := range sumTests {
		actual := functions.Add(tt.a, tt.b)
		if actual != tt.expected {
			t.Errorf("Add(%d, %d) = %d; expected %d", tt.a, tt.b, actual, tt.expected)
		}
	}
}
```

相关阅读：

[单元测试](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter09/09.1.html)  
[基准测试](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter09/09.2.html)  
[Go package - testing](https://golang.org/pkg/testing/)  

## text

主要是关于文本分析解析的一些包，但又不同于字符串处理，主要涉及词法分析 scanner、模板引擎 template、tab 处理 tabwriter。

text/scanner，主要是做词法分析的，如果大家读过我的专栏翻译的几篇关于词法分析的文章，对它的理解会比较轻松。

text/template，用于文本的模板处理，相对于 html/template 的具体应用场景，text/template 更通用。要熟悉使用它，还需要掌握它的一些方法，比如 Action、Argument、Pipeline、Variable、Function。

text/tabwriter，感觉没啥介绍的，好像主要是根据 tab 进行文本对齐的。

相关阅读：

[text/template](https://golang.org/pkg/text/template/)
[A look at Go lexer/scanner packages](https://arslan.io/2015/10/12/a-look-at-go-lexerscanner-packages/)  
[Go 模板嵌套最佳实践](https://colobu.com/2016/10/09/Go-embedded-template-best-practices/)  
[Package tabwriter](https://golang.org/pkg/text/tabwriter/)

## time

关于日期时间的包，Go 中的 unix timestamp 是 int64，表示的时间范围相应的也就有所扩大。其他的诸如睡眠、时区、定时控制等等都支持，Go 中有个逆人性的规则，那就是日期时间的格式化字符，比如传统语言的格式化字符串 YYYY-MM-DD 在 Go 却是 2006-01-02 的形式，奇葩不奇葩。

相关阅读：

[Go 标准库--time 常用类型和方法](https://shockerli.net/post/golang-pkg-time/)  
[Go 时间、时区、格式的使用](https://blog.csdn.net/qq_26981997/article/details/53454606)  
[golang package time 用法详解](https://juejin.im/post/5ae32a8651882567105f7dd3)  

## unicode

unicode 编码相关的一些基本函数，读源码会发现，它通过把不同分类字符分别到不同的 RangeTable 中，实现提供函数判断字符类型，比如是否是控制字符、是否是字母等。另外两个包 unicode/utf8 和 unicode/utf16 可用于 unicode (rune) 与 utf8 (byte)、unicode (rune) 与 utf16 (int16) 之间的转化。

相关阅读：

[go package 之 unicode](https://golang.org/pkg/unicode/)
[Unicode 码点、UTF-8/16编码](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter02/02.5.html)

## unsafe

Go 语言限制了一些可能导致程序运行出错的用法，通过编译器就可以检查出这些问题。当然，也有部分问题是无法在编译时发现的，Go 给了比较优化的提示。但通过 unsafe 中提供的一些方法，我们可以完全突破这一层限制，从包名就可以知道，unsafe 中包含了一些不安全的操作，更加偏向于底层。一些比较低级的包会调用它，比如 runtime、os、syscall 等，它们都是和操作系统密切相关的。我们最好少用 unsafe，因为使用了它就不一定能保证程序的可移植性或未来的兼容性问题。

相关阅读：

[Go 圣经 - 底层编程](https://books.studygolang.com/gopl-zh/ch13/ch13.html)  
[Go package - unsafe](https://golang.org/pkg/unsafe/)

## vendor

标准库中依赖的第三方包，当然也都由 Go 官方所开发，默认包括的依赖有：

- golang_org/x/crypto
- golang_org/x/net
- golang_org/x/text

举个例子，加密相关的 crypto 包中实现就用到了 `golang_org/x/crypto/curve25519` 中的方法。

除了内置标准库，官方还提供了其他很多库，诸如 crypto、net、text 之类的包。具体可查看 Go 官方 [github 地址](https://github.com/golang)。

博文地址：[如何阅读 Go 源码](https://www.poloxue.com/posts/2019-08-15-how-to-read-golang-source-code/)

