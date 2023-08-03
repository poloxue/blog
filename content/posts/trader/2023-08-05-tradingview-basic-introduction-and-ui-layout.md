---
title: "TradingView 基础入门"
date: 2023-08-01T17:29:19+08:00
draft: true
tags: ["tradingview"]
comment: true
description: "本文是 TradingView （以下简称 TV）教程的起始篇，将主要聚焦于 TV 的基础使用。"
---

本文是 TradingView （以下简称 TV）教程的起始篇，将主要聚焦于 TV 的基础使用。

# 基本介绍

TV 是一款强大的交易行情图表分析软件，特别受到技术分析爱好者的喜爱。同时，它也是一个交流平台，全球的交易员都会在这个平台发布自己的观点，不论国家地域。如果想找策略想法，这里是一个不错的去处。但在学习别人的思路方法，还是要独立思考，不要盲从。

一起先看看它的首页，访问 [www.tradingview.com](https://www.tradingview.com) 打开 TV 首页，如果是已注册的用户，将映入如下内容：


![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-01.png)

> 在正式开始前，如果你还没有 TV 账户，可以用如下链接注册：[邀请注册链接](https://www.tradingview.com/gopro/?share_your_love=poloxue123)。
>
> 有些人可能是直接下载使用 TV 的 Desktop APP，很少进入主站。如果想快速熟悉一款工具，主站布局可以看看，它能帮助我们相对全面了解一款工具的能力图谱。

从上图看出，TV 提供的交易品种非常丰富，涉及了全球大部分市场，诸如美股、A 股、期货、外汇、债券和加密货币等品种。

于我们而言，一般使用它的图表 - SuperCharts。查看头部的 "Products"，显示它的产品有 supercharts 图表、Screeners 筛选器、Heatmap 热力图和 Calendar 日历。

还有，通过社区 Community 板块，你还能看到不同人的分析观点，文章或者交易策略，甚至是别人分享的一些个人研究的策略。里面一些的 idea，可以作为我们的交易策略的思路来源，不过一切还是要基于个人的分析回测的基础上，甄别适合你的策略。

访问 [supercharts](https://www.tradingview.com/chart/) 地址进入图表，如果还没有注册个人账号，默认内容如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-02.png)

如上图所示，从主界面上观察，TV 图表主要有五个大的部分组成，分别是基础和作图工具、图表区域等，四和五区域无法用一两个词来概述，等下面具体详细叙述。

# 区域 1 基础工具

这个区域的主要是一些基础的工具，如用于选择标的、时间周期、技术指标、价格线样式和指标，同时设置 Alert 警报与实现 Replay 重放等。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-03.png)

选择标的是上图的 AAPL 区域，TV 支持的品种丰富，点击即可进行 Symbol Search，快速发现你关注的标的。以搜索 MSFT 为例，效果图如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-04.png)

展示的搜索结果外，还能看到 TV 支持的交易品种，包括 Stocks（股票）、Funds（基金）、Futures（期货）、Forex（外汇）、Crypto（加密货币）、Bonds（债券）等；

这部分还隐藏了一些高级功能，如多品种的 Compare，提供运算功能实现多品种指数 Index 的生成，是属于高级些的功能，有机会说到多品种组合交易或套利交易的时候再聊。

## 时间周期

时间默认是 D 级别，即日线级别。TV 默认提供的时间周期有 1m、3m、5m、15m、30m、1h、2h、4h、6h、12h、D、W、M 等。如果是用户购买了会员，还可以使用它的秒级别数据线。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-05.gif)

TV 本身并没有把限制时间周期的选项，我们可以选择分钟级别以上任意的时间周期，如 16m、100m 等等。

如上动图中新增了一个周期选项 100m。且当处于界面时，以数字开头输入，也可以选择任意的时间周期，非常便捷。图中，我输入 "150" 快速跳转到了周期 150m 的蜡烛线。

## 指标

指标从来源上分类有系统内建指标、社区指标和自建指标。点开 "Indicators"，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-06.png)

含义如下：

- Favorites，即我的收藏，当看到个人钟爱的指标，存放收藏的指标；
- My scripts，是自建指标，通过 pinescript 开发自建的指标；
- Technicals，即技术指标，系统内建的技术指标，如常见的 MA（Moving Average）、RSI（Relateive Strength Index）等；
- Financials，即财务数据，如资产负载（balance sheet）、损益表（Income Statement）和现金流表（Cash Flow Statement）；
- Community Scripts，是社区脚本，社区用户通过 Pinescript 自建指标分享给社区，其他人可查看使用。

> 说明，TV 除了常见的技术指标，另一个优势在于用户能利用 pine script 基于个人的思路创建新的指标。而且，可利用产生的信号进行快速的回测，也是基于这点，这里 Indicators 下是有 "Indicator" 和 "Strategy" 两个分类。

## 价格线

价格线样式默认是蜡烛线，TV 提供的价格线支持样式种类丰富，如 Bars、蜡烛线（K 线）、Heikin Ashi、Hollow candles、Line、Renko、Kagi 等等。每种线样式都有各自不同的生成规则。常用的主要是蜡烛线。其他类型可自行查阅资料，这里不多解释了。

## 报警设置

TV 支持警报通知的设置，即菜单中的 Alert。借助这个能力，可设置基于价格、指标或趋势线的报警功能，如 BTC 价格突破关键点位 20000，趋势线突破，或者指标金叉死叉等。点击 Alert，配置页面如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-07.png)

Settings 是报警的规则配置，如图中的价格的阈值报警规则的设置界面。Notifications 是设置通知的规则，如是 app 通知（TV 有 mobile app）、弹框、邮件通知、声音提示、发送邮件等，此外，对于 pro 及以上用户，还支持 webhook，基于它实现实盘交易。

## 重放功能

重放功能，即菜单中的 Replay，使人工回测想法成为现实，训练自己操盘能力的工具。遗憾的是，普通用户只能进行日线级别的重放，好像还有 bar 的数量限制。

效果图如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/tradingview-basic-introduction-and-ui-layout-08.gif)

回测速度的快慢是可控的，也可将重放暂时，仔细分析盘面再执行下一步的操作。

> 补充，除了提供 Replay 手动回测，TV 还支持针对自动化量化回测的 Pine script，可快速测试诸如 Moving Average ，RSI 等常见策略。

# 区域 2 画图工具

画图工具位于界面布局的左侧栏，可以看到，TV 提供了丰富的作图工具，如基本的趋势线就有很多分类，其他还有如与斐波拉契相关的工具、盈亏比分析工具、形态识别。

# 区域 3 图表区域

这部分是图表区域，价格线、指标

# 区域 4 标的、报警列表、新闻等

# 区域 5 选股、脚本编辑回测与交易面板

这个区域就叫

# 参考资料

- [The Ultimate TRADINGVIEW TUTORIAL for BEGINNERS in 2023 | Step by Step Guide - YouTube](https://www.youtube.com/watch?v=pWF3AkPzLrE)
- [A Beginner’s Guide to TradingView | Binance Academy](https://academy.binance.com/en/articles/a-beginner-s-guide-to-tradingview)
- [How to Use TradingView Drawing Tool](https://www.youtube.com/watch?v=W-_b0wuRahs)
- [How to Draw on TradingView: A Comprehensive Guide](https://www.financialtechwiz.com/post/how-to-draw-on-tradingview)

