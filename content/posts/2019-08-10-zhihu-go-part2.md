---
title: "Go 问答汇总 Part Two"
date: 2019-08-10T15:38:23+08:00
draft: false
comment: true
tags: ["Golang"]
---

继上篇 [Go 问答汇总](https://www.poloxue.com/posts/2019-07-22-zhihu-go-part1/)，已经过去了一个多月。今天汇总下近一个多月我关于 Go 的回答。

粗略数了一下，一个多月的时间里，大约回答了 18 个与 Go 有关的问题，问题主要是来源于 segmentfault 和 zhihu 两个平台。后面希望加入更多平台，如 stackoverflow、github 的感兴趣主题。

最近在写一个小工具，准备用于帮助自己回答不同平台的问题，同时也便于每个月的问题汇总。写的有点慢，希望月底可以完成。

正文部分开始。

---

[golang中如何将redis取出的map[string]string数据解析到目标struct中？](https://www.zhihu.com/question/332744634/answer/736204517)

主要和反射相关。

问题主要是关于 map 中如果存在日期字符格式串，如何解析到 time.Time 类型成员中，而对于结构体而言，reflect.Kind() 返回的只能说明字段类型是 struct，并不能确定真正的类型，这时可以用 Go 的 switch type 类型查询语法实现。

补充一点，在回答中没有提到的。

在实现 map 到 struct 的通用方法时，我们比较容易想到支持基础类型，但对于结构体类型而言，可能性太多，如何更灵活地解决问题？我觉得，可通过钩子方式实现，即如果自定义类型需要支持 map 到 struct 的转化，可通过在自定义类型上增加钩子方法实现，比如 UnmarshalMap。如何实现可参考下 encoding/json。

当然，这个工作已经有人做了，参考 github 上的包，[mitchellh/mapstructure](https://github.com/mitchellh/mapstructure)。前面说的 Hook 也是支持的。

[golang 怎么优雅的实现错误码？](https://segmentfault.com/q/1010000019676525/a-1020000019948975)

Go 对错误处理有一套自己的理念。这个问题，我只是简单回答了一下，简单的思路，我定义了用户级别错误和系统级别错误。上篇[问答汇总](https://zhuanlan.zhihu.com/p/71659098)也会类似问题。

[golang什么时候该返回error，什么时候panic？](https://segmentfault.com/q/1010000020000806/a-1020000020006211)

我的建议是，发生的 error 是否已经严重影响服务逻辑，如果在预判之内的错误，我们就应该 return error，记录日志，并不需要人工干预才能恢复，否则建议 panic。

举个例子，在一般情况下，服务启动时，需要进行完善的初始化工作，确认各个组件的运行正常。如果初始化都失败了，那就没有必要继续向下走了，应该 panic 赶紧提示。

[Golang time 如何实现的？](https://www.zhihu.com/question/320347209/answer/736754990)

问题标题看着挺大，其实题主关心是几个核心常量之间的转化关系。主要是三个时间，分别 unix 时间、wall 时间和 absolute 时间。这里面有个相对重要的转化公式，在需要考虑平润年的时候稍微有点复杂。

不多介绍了，具体自己看回答吧。

[golang 中时候用指针什么时候用普通对象？](https://segmentfault.com/q/1010000019708895/a-1020000019911459)

其实就两点，一是如果数据结构比较大，建议采用指针，不会发生值拷贝。二是如果需要修改结构的话，必须用指针。当然如果是引用类型，比如 chan、slice、 map，就不用考虑这个问题了。

[Golang中的make(T, args)为什么返回T而不是*T?](https://www.zhihu.com/question/312356800/answer/739572672)

make 针对的是 Go 的引用类型，即 chan、slice 和 map，而 new 针对的指针。引用类型为什么 make 不是返回指针呢？这样一说好像和上个问题有点类似了，当然因为指针不存在值类型的那些问题。

[在循环中 append map 到 map slice，map slice 中的数据全部为最后一次 append 的数据](https://segmentfault.com/q/1010000019881129/a-1020000019949131)

与上一个问题知识点类似，map 是引用类型，即使 slice 通过 append 赋值了多份 map 变量，但是其内部指向是同一个地址。

[golang中哪些引用类型的指针在声明时不用加&号，哪些在函数定义的形参和返回值类型中不用*号标注](https://segmentfault.com/q/1010000019965306/a-1020000019996800)

与前面问题类似，具体看回答。

我的理解，从这里向前数的四个问题，考察知识点基本类似，简单点说，就是 make 和 new 和问题。本质上讲，就是变量内部就是什么的问题。

[为什么 go语言的slice内部函数那么少？](https://segmentfault.com/q/1010000019341665/a-1020000019911483)

说实话，我也不明白为什么 Go 团队没有像为 string 类型那样为 slice 提供相应的标准库帮助 slice 更方便的操作。但其实，即使没有这样的包， slice 常见的各种增删改查也是可以实现，就是稍微有点 hack。具体如何实现，看看我的问答吧。

[golang 等值比较是不是直接比较地址呢？](https://segmentfault.com/q/1010000019940462/a-1020000019941598)

首先要说 Go 的等值比较比较的是值，而不是地址。Go 中变量的可比较类型是内置的，基本所有类型都可以进行比较，包括 interface 和 struct。两个变量可比较的提前必须是相同类型。但有一点需要说明的是，interface 是不确定的类型，所有它不但会比较值，还有比较具体的类型。 

回答完这个问题后，我突然想起前段时间比较两个同类型结构体时还用了反射包中 reflect.DeepEqual 方法，真的是浪费资源啊。

[golang 中如何禁止一个导出类型直接构造，必须通过new函数来构造？](https://www.zhihu.com/question/333771024/answer/741498087)

其他的 oo 语言实现题主要求是非常简单的，只要定义相应的私有成员属性并通过构造函数控制输入的参数即可。

那么 Go 该如何实现呢？其实也很简单，思路与 oo 是类似的。只是我们把 oo 语言中的构造函数换成了 Go 中的工厂方法，私有变量变成了 Go 包级别的私有成员属性。我们只需要通过定义指定的可导出的工厂方法创建实例即可。

[入门，进阶GO语言，有什么好的书籍推荐？](https://www.zhihu.com/question/333677492/answer/743612095)

我从入门、中级到进阶三个阶段推荐了几本书。有兴趣的朋友，具体查看回答吧。

[如何评价 Go 标准库中新增的 plugin 包？](https://www.zhihu.com/question/51650593/answer/747619907)

这个问题的回答，我是先学先买的。看了 medium 中几篇关于 plugin 使用案例的文章，总共花了大概三四小时。plugin 包使 Go 是可以实现动态模块加载的能力，可以在不用重新编译主程序的情况下加入新功能。这是有一定的价值的。

但 plugin 包也存在一些问题，使用起来会用一些限制因素。但如果我们清楚地了解，还是能拎的清我们应该在什么场景下使用它。具体有啥限制，查看回答吧。

[go build 如何隐藏全局静态字符串变量？](https://segmentfault.com/q/1010000019941363/a-1020000019955654)

这个问题，我并没有找到啥好办法，能想到的就是通过加密解密的方式解决。当然，其实在真实的项目中，我们可以通过引入外部服务实现，比如 k8s 的配置加密功能，使用 ectd 管理配置等。

[go语言中, 空的死循环与永远阻塞的chan细节上有什么差异?](https://www.zhihu.com/question/39766078/answer/748425459)

Goroutine 是抢占式的，这不同于传统的协程模型。但它又不是完全的抢占式，单核的情况下，还是需要 CPU 主动出入资源的，而空死循环将会一直占用着 CPU，对资源的浪费严重，而 chan 阻塞会出让 CPU 资源，实现并发执行。这应该是两者最大的不同吧。

除了一般的 chan 实现阻塞，问答还介绍一些其他方式，也可以实现类似的效果。感兴趣的朋友可查看回答。

[Golang中fmt.Println和直接println有什么区别？](https://www.zhihu.com/question/335186436/answer/750846252)

println 主要是 Go 自己使用，比如源码、标准库等，而 fmt 才是给 Go 开发人员使用的。而且要提的是 println 不能保证兼容性，可能在未来的某一天就不存在了，但 fmt 中的函数就不存在着这样的问题。

当然，两者的使用和效果上也是有区别的，如 println 输出是到标准错误的，而非标准输出。

[如何阅读Golang的源码？](https://www.zhihu.com/question/327615791/answer/756625130)

这个回答是个大工程，零零碎碎花了我差不多三个星期的时间。什么原因呢？

我提了一个阅读源码的思路，分成了大概三步，大致分别是，了解使用、熟悉架构和剖析原理。我最近在思考是否自己也开始剖析源码，于是决定先以第一步入手，把 Go 的源码整个撸一篇，大概了解涉及了的内容和它们的用途。总共涉及四十多个部分，于是花的时间就比较长。

第一步，了解使用这一块，有些部分达到了解使用，但是有些部分只是达到了解的层次。如果要完整阅读源码，很难，但大概阅读还是可行的。准备等等合适时机启动系统的源码阅读的计划。


[golang数据库操作的时候，需要go func()吗？跟python异步操作yield有什么不同？](https://www.zhihu.com/question/339444132/answer/782537354)

上篇 [Go 问答汇总篇](https://zhuanlan.zhihu.com/p/71659098) 也有类似问题。我的理解，使用 go func 的前提是必须有可并行执行的任务，这是一个重要前提。 

很多时候，大家学 Go 都是冲着 Go 的并发来的，结果学之后发现压根用不上，很郁闷啊，总觉得哪里用的不对，不应该是这样的。一般的业务开发，特别是 web 的业务开发，使用并发的场景确实不多。而并发性能的问题，服务框架已经帮助我们实现了，完全没有插手机会，要想真正学会并发，不能只是每天的增删改查。

汇总完毕！回答如有错误，欢迎大家指正。最后，感谢阅读，汇总篇似乎是比较枯燥的！

