---
title: "Go 问答汇总 Part Three"
date: 2019-09-10T15:47:12+08:00
draft: false
comment: true
tags: ["Golang"]
---

第三篇 Go 问答总结，2019 年 8 月份总结，大约有 12 篇问答。前两遍地址如下：

[Go 问答汇总 Part One](https://www.poloxue.com/posts/2019-07-22-zhihu-go-part1/)
[Go 问答汇总 Part Two](https://www.poloxue.com/posts/2019-08-10-zhihu-go-part2/)

问题大部分是来自于知乎和 segmentfault。本月有一个问题来自 stackoverflow，我的英文水平一般，读与翻译还行，但写起来还需要锻炼。虽然这一个回答没得到一个赞同，但能被题主采纳，我还是很荣幸的。

最近发现，我的回答经常会被 Go 语言中文网的周刊收录。对 Go 感兴趣的朋友可以关注下 Go 语言中文网的公众号，内容还是非常丰富的，每天都会推送关于 Go 的优秀文章。

![](https://blogimg.poloxue.com/0000-wechat-03-studygolang.jpeg)

开始正文！

[dynamodbattribute.UnmarshalMap canges the type of my variable to map[string]interface{}](https://stackoverflow.com/questions/57567917/dynamodbattribute-unmarshalmap-canges-the-type-of-my-variable-to-mapstringinte/57568211#57568211)

将 stackoverflow 的这篇回答放在首位吧，问题不是很难，重在第一次尝试，stackoverflow 上面有价值的问题还是很多的。

题主的目标是希望 map 类型转化成 struct 类型。将问题稍微简化下，题主希望通过类似如下这种写法将 map[string]interface{} 转化为 user 类型。

```go
package main

import "fmt"

type User struct {
	Name string
}

func Item() interface{} {
	return User{}
}

func ItemMap(s map[string]interface{}, item *interface{}) {
	*item = s
}

func main() {
	m := Item()
	fmt.Printf("%T\n", m)
	fmt.Printf("%v\n", m)

	s := map[string]interface{}{
		"Name": "poloxue",
	}
	ItemMap(s, &m)

	fmt.Printf("%T\n", m)
	fmt.Printf("%v\n", m)
}
```

首先，通过 Item() 方法创建一个 User 类型变量，但返回的类型是 interface{}，因而 main 主函数中的 m 通过类型推导，也就是 interface{} 类型，在 ItemMap 函数中将 map[string]interface{} 变量赋值给 m 的地址并不会有任何问题，因为虽然此时 m 类型的底层类型是 User，但赋值时并不会验查到这一层，经过一部分之后，m 的底层类型就由 User 转化成了 map[string]interface，并非完成了 map[string]interface{} 到 struct 的转化。

看样子，map 和 struct 的转化还是很常见的需求啊，上月的问答总结就有类似的问题。再次推荐 mapstructure 包。

[go源码的raceenable是做什么的，可否解答一二！](https://segmentfault.com/q/1010000020049743/a-1020000020165822)

Go 是内置并发支持的语言，并发的常见问题就是出现竞态条件，Go 专门提供了一个工具用于检测这个问题，使用非常方便，只需要在执行程序时加上 -race 选项，Go 源码中的这个 raceenable 表示的意思即使是否启用了 raceenable。

前段时间翻译了一篇关于 go race 的文章，访问 [Go 译文之竞态检测器 race](https://juejin.im/post/5d5851aee51d4561c6784079)。

[Golang中，runtime.Caller(skip)，为什么会保留编译器变量？](https://segmentfault.com/q/1010000020145507/a-1020000020146489)

题主的目标是获取 Go 可执行文件的当前路径，但对于很多习惯了使用解释型语言的朋友，养成了源码文件路径即可执行文件路径的思维，而 runtine.Caller 可用于查看调用栈信息，正好可以保留编译时的源码文件信息。

但真实的项目部署的肯定是可执行文件，而非源码文件，而路径也是针对可执行文件的路径。如何获取到呢？请看回答吧！

再提醒一点，正因为很多朋友养成了解释型语言的思维，常会通过 go run 执行源码，然后获取可执行文件路径，但最终得到的确实一个看不懂的路径。这个问题在回答也有介绍。

[golang如何复用mysql连接？](https://www.zhihu.com/question/341035336/answer/793308659)

golang 中的标准库 database/sql 已经提供了一份数据库管理的公共实现，同样涉及 mysql 连接复用的功能。

题主的问题是为什么没有达到复用的效果。

连接复用有一个重要前提就是资源使用结束后要记得释放，否则将会一直处于占用状态，至于有哪些情况会导致连接一直复用，查看回答！

回答中提到了一篇非常系统介绍 Go 中 database/sql 使用的教程，有兴趣可以仔细读读。

[goroutine 出现异常](https://segmentfault.com/q/1010000020100276/a-1020000020103898)

这个问题稍微有点复杂。异常的原因，简单说来是 channel 发送者发送等待时间太长，导致异常出现。解析问题的核心思路在于，在发送者和接收之间增加一个 buffer，无论发送者发送了什么内容，先将消息接收到 buffer 中，待有空闲的接收者时将消息从 buffer 中发送出去。

题主是通过 chan 自带的 buffer 实现，创建了缓冲大小 50 的 chan。当然，更灵活的方式，还是自己实现 buffer 比较好。而且题主的场景是爬虫，可能对 buffer 的大小要求会比较高，可能适当考虑接入外部服务。

[go的reflect是否可以获取到同一个package中，其他文件定义的类型的私有方法的名称？](https://www.zhihu.com/question/340587967/answer/791508607)

首先，要明确的是，Go 中的反射是无法看到类型的私有方法的。如果想达到这个目标，只能将方法定义为可导出方法。但私有方法也有它的好处，那就是外界无法使用这个方法。那将方法定义为公开可导出是否也可以做到让外界无法使用？

查看我的回答吧！

[beego 的缓存如何转结构体呢?](https://segmentfault.com/q/1010000020063647/a-1020000020068775)

beego 中缓存的存储与获取的问题，很简单，没什么好说的！

[go语言里的select监听到底是怎么工作的?](https://www.zhihu.com/question/340342212/answer/795661248)

问题的核心不在于 select 的工作机制，而是关于 Go 中 timer 定时器的使用问题。以下是问题的核心代码。

```go
for {
	select {
		case num := <-ch:
			fmt.Println("get num is ", num)
		case <-time.After(2 * time.Second):
			fmt.Println("time's up!!!")
			//quit <- true
		}
	}
}()
```

题主的问题是为什么 timer 的 case 一直没有执行。这是因为 time.After 在每次循环都会创建的 chan，之前定时 chan 已经不再监听了。

我开始觉得，这种方式挺适合处理任务出现 timeout 的情况，经过一位朋友提醒，这里面存在着泄露风险。具体原因，阅读回答了解吧！

[在golang的设计里，为什么不能用switch实现select的功能？](https://www.zhihu.com/question/27170192/answer/808995623)

主要是场景决定吧，channel 监听有点类似于异步 IO 的多路复用，select 可以说是多路复用的专用名字了，switch 是分支结构，硬要加上其他功能，也不便于理解，我的感觉是，独立出一个新的关键字会更好理解。

[go语言struct中有函数指数的示例讲解](https://segmentfault.com/q/1010000020110214/a-1020000020110738)

题主要求用从浅入深的方式介绍，Go 中 struct 含有函数成员的问题。我觉得我也写得还不多，但是没有得到题主的采纳，只得到一个评论夸赞。

[golang 编译器分32和64版本吗？](https://segmentfault.com/q/1010000020128222/a-1020000020130806)

肯定是分的，但我们不用关心，go 命令已经封装好了，它会依据平台选择不同的底层命令。

在 go 安装目录的 pkg 目录下有个 tools 目录，里面包含了编译链接时实际使用的命令，比如我的 Mac Pro，在 pkg/tool/darwin_amd64/ 下能找到 go 编译链接实际调用的命令 compile 和 link。darwin_amd64 中 drawin 表示操作系统，amd64 就是系统架构。

执行 go env 会发现其中有个环境变量 GOTOOLDIR，它是那些与我们平台相关工具的所在目录。我的 Mac 下得到的结果是：

```
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
```

[正则如何匹配最后一个页码?](https://segmentfault.com/q/1010000020244344/a-1020000020246932)

题主的问题是如何进行 html 的解析。

```html
<ul id="pages">
<li><a href="xxx1">1</a></li>
<li><a href="xxx2">2</a></li>
<li><a href="xxx3">3</a></li>
</ul>
```

这类问题一般是不推荐使用正则匹配，可以试试 goquery 这个库，它的用法类似 jquery，可用于解析 html 内容，使用非常方便。当然，回答中也介绍了正则的实现，不过代码的可读性看起来比较差。

本月的回答总结完毕，如果大家发现有不当的内容，欢迎批评指正，非常感谢。

博文地址：[Go 问答汇总 Part Three](https://www.poloxue.com/2019-09-10-zhihu-golang-part3.md)

