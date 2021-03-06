---
title: "为什么要学 Go"
date: 2019-04-08T17:31:32+08:00
draft: false
comment: true
tags: ["golang"]
---

新学一门语言，大家都想先弄清楚为什么要学它？玩知乎一段时间更是让我感受深刻，诸如

- 为什么要学习Python？
- 为什么要学习C？
- 为什么要学习Java？

之类问题经常出现在眼前。以前学语言时倒没怎么关心过这类问题。今年公司由于新业务需要开始全面从PHP转型到Golang。所以我学习它也就是为了工资。额？不能这么俗气，还是具体想想自己为什么要学习Golang吧。

作为一名golang新人，在写这篇文章时我搜罗到不少golang的优秀资料，在文章最后分享出来。

## 大势所趋

趋势如此，这应该是多数朋友开始学习它的原因。追涨杀跌，这是大多数人喜欢的操作手法。

何以证明这个趋势呢？

首先，我的亲生经历是听到看到golang这个词的频率越来越高，不过，这个太难量化了。来介绍一款工具，google trend，即google趋势。它是google利用自身优势，通过对搜索关键词进行统计分析，根据单词频率分析特定时期某类事物发展趋势的一款分析工具。

我们可以用 google 趋势来分析一下近年来 golang 的发展趋势，点击链接。

先看看时间线上的表现，历史的变化趋势：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-04-08-why-learn-go-01.png)

可以看出，从2015年到2019年golang的发展趋势一直处在稳定上升阶段；

不过我们会想，这只能说明golang在世界上整体趋势表现较好，但在中国是否一样火热。这个大可不必担心，google趋势中也有区域的统计信息：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-04-08-why-learn-go-02.png)

可以看出，Golang在世界区域的分布情况，前五名分别是，中国、新加坡、圣赫勒拿、韩国、香港。其中，Golang在中国的流行程度简直就是一骑绝尘、遥遥领先。

注：如果想分析中国各城市的表现情况，可以点击地图就可进入特定国家进行分析。

除了google趋势，还可以来看看在TIOBE语言排行榜上的表现。点击链接

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-04-08-why-learn-go-03.png)

额？怎么才十六名，好紧张、好难过，难道学错语言了吗？不对，得找几个理由安慰下自己。

Golang是一门非常年轻的语言，仅用十年时间就从世界上数以千计的编程语言中脱颖而出，发展速度迅猛。诸如Java、Python、PHP、Javascript都和我一样处在了奔三的路上，近30载的发展才有当前的生态与地位；

Golang在2018年的最好成绩曾到达过前十名，这个成绩足以说明golang的流行程度。而且排名存在浮动也是很正常的事情，Golang这些年稳步的发展趋势还不能给我们足够的信心吗？

通过以上的数据分析，我们得到了一些结论，不过感觉说服力不足，有种空喊口号 "我们能赢" 的感觉。趋势很好，就认为稳赢，显然这是很不合理的。所以，我们还需要分析一些更层次的原因。

## 核心成员

为什么要了解核心成员呢？核心成员某种意义上是语言的招牌。就像投资，肯定选择相信巴菲特，而不是你。

Golang的核心开发组成员由一群大神级人物组成。其中，最核心的三人分别是Ken Thompson、Rob Pike、Robert Griesemer。

Robert Griesemer，参与开发了 Java HotSpot 虚拟机和Javascript的Chrome V8引擎；

Ken Thompson，C和B语言的设计者、Unix创始人之一，操作系统Plan 9的主要作者，1983年图灵奖得主；

Rob Pike，UTF8的主要设计者，与Ken Tompson为贝尔实验室的同事，共同参与了Plan9。而且Golang的logo，据说是囊地鼠，英文gopher，就是Rob Pike的妻子设计的；

都是如此这般牛人坐镇，可见golang的层次已经高出其他语言很多个台阶了。

## 背景历史

清楚它的产生背景与发展历史，才能更好了解它的特性与使用场景。

首先，Golang诞生于google。有了大厂庇护，才好开挂。google曾经一直有个传统，允许员工自由支配本属于工作时间的20%来用于创新实践，这为google带来很多开创性的项目，其中就包括Golang。但听说，前几年该传统已经被取消了。

Golang早起的讨论由前面介绍的三位大牛发起，针对性分析了当时的环境背景。

首先，当时传统的编程语言通常都会有如下一些缺点：

- 学习成本太高，如C++，为准确表达作者思想，我们要花费大量时间学习语言；
- 编译速度太慢，代码的编写、预处理、编译与运行流程花费时间太长；
- 缺乏类型检查，主要指诸如python、php等解释性语言，这常会导致一些低级错误发生；
- 而且计算机领域相比于前些年也发生了很多变化，比如：
- 计算机硬件发展迅速，软件已经不能充分发挥它们的优势，比如多CPU；
- 语言越来越复杂，要么并发与性能不佳，要么风格不够优雅且不统一；
- 人力成本越高越贵，项目的迭代周期越来越短；

针对如上的各种情况，于是在2007年，他们正式开始着手Golang的设计与开发，并在2009年的11月正式发布。我们列举下，接下来一段时间，Golang发展中几个关键节点。

- 2012年3月，正式发布1.0版，走向成熟；
- 2015年8月，发布了1.5版，实现自编译，移除最后残余的 "C代码"；

更新迭代速度多，基本保持了每半年更新一个版本；

- 2017年2月，发布1.8版
- 2017年8月，发布1.9版
- 2018年2月，发布1.10版
- 2018年8月，发布1.11版
- 2019年2月，发布1.12版

如此给力的团队与稳定的版本迭代速度，某种程度也促成了golang快速发展。

## 语言特性

各种介绍go的文章讲的最多的两点特性：静态语言与高并发。但是取交集的话，特性就太少了。介绍细致些吧，但如此一来，这段就显得很是无聊，

了解golang特性前，可先来看看它的几个设计原则。在网上搜罗了些资料，总结出大概几点：

- 大道至简，比如及其简单但完备的面向对象设计，面向接口，没有继承只有组合；
- 最少特性，一个特性对解决问题有显著效果就没有必要存在；
- 显式表达，比如数据类型必须显式转化，不提供隐式转化能力；
- 最少惊异，减少那些奇怪的特性设计，最大程度减少错误发生概率；

从产生背景我们可以知道，Golang在主要针对其他语言痛点而设计的。它有哪些特性？

- 静态语言、静态编译速度快，拥有静态语言的安全与性能；
- 天然支持并发，基于CPS并发模型，goroutine轻量级线程，支持大并发处理；
- 简洁的脚本化语法，如变量赋值 a := 1，兼具静态语言的特性与动态语言的开发效率；
- 提供垃圾回收机制，不需要开发人员管理，通过循环判活，再启用goroutine来清理内存；
- 创新的异常处理机制，普通异常通过返回error对象处理，严重异常由panic、recover处理；
- 函数多返回值，方便接收多值，一些解释性语言已经支持，如python、js的es6等；
- 支持defer延迟调用，从而提供了一种方式来方便资源清理，降低资源泄露的概率；
- 面向接口的oop，没有对象与继承，强调组合，以类似于duck-typing的方式编写面向对象；

那么多特性，好无聊，不对，应该是好厉害。之前在知乎上看到过有位朋友写了个十分钟GO快速入门的文章，挺有趣的，分享出来。看过之后应该对上面这些特性有更直观的认知。

知乎地址在 [GO十分钟快速入门](https://zhuanlan.zhihu.com/p/28527216)，代码在 [GO Play 代码体验](https://play.golang.org/p/tnWMjr16Mm)。

## 优秀项目

已经说了那么多Golang的牛x之处，但以前出现过的很多语言也都是这么宣传的。
语言的目标是用于项目开发，并能打造出很多优秀的产品。那么，Golang有哪些好像优秀的项目呢？不搜不知道，一搜吓一跳！列举一下我收集到的golang开发的优秀项目，
如下：

- docker，golang头号优秀项目，通过虚拟化技术实现的操作系统与应用的隔离，也称为容器；
- kubernetes，由google开发，简称k8s，k8s和docker是当前容器化技术的重要基础设施；
- etcd，一种可靠的分布式KV存储系统，有点类似于zookeeper，可用于快速的云配置；
- codis，由国人开发提供的一套优秀的redis分布式解决方案；
- tidb，国内PingCAP 团队开发的一个分布式SQL 数据库，国内很多互联网公司在使用；
- influxdb，时序型DB，着力于高性能查询与存储时序型数据，常用于系统监控与金融领域；
- cockroachdb，云原生分布式数据库，继NoSQL之后出现的新的概念，称为NewSQL数据库；
- beego，国人开发的一款及其轻量级、高可伸缩性和高性能的web应用框架；
- caddy，类比于nginx，一款开源的，支持HTTP/2的 Web 服务端；
- flynn，一款开源的paas平台；
- consul，HashiCorp公司推出的开源工具，用于实现分布式系统的服务发现与配置；
- go-kit，Golang相关的微服务框架，这类框架还有go-micro、typthon；
- go-ethereum，官方开发的以太坊协议实现；
- couchbase，是一个非关系型数据库；
- nsq，一款高性能、高可用消息队列系统，每天能处理数十亿条的消息；
- packer，一款用来生成不同平台的镜像文件的工具，例如VM、vbox、AWS等；
- doozer：高速的分布式数据同步服务，类似ZooKeeper；
- tsuru：开源的PAAS平台，和SAE实现的功能一模一样；
- gor：一款用Go语言实现的简单的http流量复制工具；

项目列举了这么多，从此也可看出现在很多新项目都在使用Golang开发，涉及到很多领域。

## 应用领域

接下来了解下Golang具体擅长哪些领域，如果不适合自己所在行业，暂时就没必要去学习了。

### 区块链

当前的两个主流区块链框架，分布式记账本框架hyperledger和以太坊合约框架go-ethereum都是使用Golang开发；下图是某招聘网站关于区块链职位要求技能的分析。


![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-04-08-why-learn-go-04.png)

### 微服务

现在越来越多的项目会采用微服务架构，前面介绍的优秀项目中也看到很多go提供的微服务框架，如git-kit、micro等。

举一些具体公司的例子，比如[今日头条使用 Golang 构建了千万级微服务](https://36kr.com/p/5073181)；

### 云服务

云服务，如国内著名的七牛云全站采用Golang开发；还有如盛大CDN、阿里云CDN等；

很多的云平台基础设施如docker、kubernetes等为Golang开发；

京东的消息推送与分布式存储也是如此；

### 分布式

诸如数据库中间件、代理服务等很多采用Golang开发，比如前面的介绍codis、cockroachdb、etcd等；

### 其他

很多领域都能看到Golang的影子，诸如直播领域、游戏开发等等，在其中golang为后台的调度系统、任务处理，批量的数据计算、系统监控等提供了各种解决方案。
比如，最近知乎近也使用Golang进行重构了自己的推荐系统。

[舍弃 Python，为什么知乎选用 Go 重构推荐系统？](https://www.infoq.cn/article/ur32PLxfWKEyDG*ZLE0f)

很多涉及领域就不一一列举了。反正一句话就是很牛。

## 学习资料

说这么多，主要是为给自己好好学习找个借口。接下来分享一些近期收集的Golang学习资料。

### Golang官网

Golang官方地址： golang.org，无论学习什么知识，第一手资料基本都是首发于官网。进入到官网后，会看到很多资源，比如：

- 文档：[golang.org/doc](https://golang.org/doc/)，官方文档，仔细读下文档首页并分类，了解下自己要学哪些内容；
- 一览：[tour.golang.org](https://tour.golang.org/welcome/1)，交互式运行环境，不安装golang便可体验学习它的语法与使用；
- 指南：[golang.org/ref/spec](https://golang.org/ref/spec)，golang学习指导手册，从基础语法到高级特性全部都有介绍；
- 标准库：[golang.org/pkg/](https://golang.org/pkg/)，可以查看所有的官方库的接口、源码以及使用介绍；
- 博客：[blog.golang.org](https://blog.golang.org/)，不定期分享go的最佳实践，有些公司也会投稿介绍自己的案例；
- 实验室：[play.golang.org](https://play.golang.org/)，感觉和tour类似，不过在这里编写的代码可以分享给别人；

等等。

官网是个宝库，我们需要认真仔细去挖掘其中的内容；但由于一些原因，golang的官方站点我们无法访问，不过golang为我们提供了中国的官网，地址：golang.google.cn；

### golang社区

一门语言的发展需要有大批牛人的分享布道，也需要我们这些菜鸟学习有更多的参考路径。这一切都离不开社区。国内外也有很多优秀的go语言社区；

- go语言中文网，[studygolang.com](https://studygolang.com/)，分享Go语言知识，聚合各种golang文章和书籍资料；
- go交流论坛，[gocn.vip](https://gocn.vip/)，go语言学习交流论坛；
- go官方讨论组，[forum/golang-nuts](https://groups.google.com/forum/#!forum/golang-nuts)，golang的官方邮件讨论组；

### 一张图、一个目录与一个合集

在整理资料时，发现太多优秀的开源项目与书籍，重复工作就不做了，分享几个别人整理的优秀资源。如下：

- 一个目录：[索引目录地址](https://github.com/Unknwon/go-study-index)，各种go语言资源的汇总；
- 一张图谱：[图谱processon地址](https://www.processon.com/view/link/5a9ba4c8e4b0a9d22eb3bdf0#map)，源自Golang Foundation；
- 一个合集：[awesome-go](https://awesome-go.com/)，其中整理了大量golang的库、框架和一些著名项目；

我的博文：[为什么要学 Go](https://www.poloxue.com/posts/2019-04-08-why-learn-go)
