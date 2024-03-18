---
title: "从零量化交易 Part 1: 基本介绍篇"
date: 2024-08-06T16:37:03+08:00
draft: true
comment: true
---

> 本系列文章目标是，基于 Python 从零实现策略回测，示例策略选用最简单的双均线策略。
>
> - [从零量化交易 Part 1: 基本介绍篇](../2023-08-06-algo-trading-from-scratch-introduction)
> - [从零量化交易 Part 2: 数据准备篇](../2023-08-12-algo-trading-from-scratch-fetching-data) (待完成)
> - [从零量化交易 Part 3: 量化策略篇](../2023-08-16-algo-trading-from-scratch-strategy-rules) (待完成)
> - [从零量化交易 Part 4: 交易信号篇](../2023-08-06-algo-trading-from-scratch-strategy-signals) (待完成)
> - [从零量化交易 Part 5: 策略回测篇](../2023-08-06-algo-trading-from-scratch-backtesting-strategy) (待完成)
> - [从零量化交易 Part 6: 收益评估篇](../2023-08-06-algo-trading-from-scratch-evaulating-returns) (待完成)
> - [从零量化交易 Part 7: 风险管理篇](../2023-08-06-algo-trading-from-scratch-risk-management) (待完成)
> - [从零量化交易 Part 8: 实盘接入篇](../2023-08-06-algo-trading-from-scratch-live-trading) (待完成)

本篇为系列文章的第一篇，介绍关于策略回测的基础内容。

# 什么是策略回测？

假设，我通过观察发现了一个可能赚钱的策略，10 日与 20 均线金叉入场，死叉出场。现在如果我在实盘中使用这个策略，它能产生好结果呢？如果不是过于自信，绝不敢轻易下这个结论。

其实，交易时经常会有想法闪现，好像发现了秘籍，但基本经历不了测试的考验，以亏损收尾。这里所说的测试即策略回测，通过把想法在历史数据上测试与分析，得出这个策略是好是坏。

在明白策略回测重要性后，究竟如何开始回测？

# 常见的回测途径

手撸前，先介绍下一般情况下策略回测的途径吧，从易到难，即可基于现成的量化平台，也可基于开源框架组装本地集成。

## 基于专业的量化平台

国内有不少专业的量化平台，有支持非编程回测方式，也有通过编程实现的平台。有一些平台有自己的特色。

从核心功能上评价，它们帮用户解决了上手难的问题，体验丝滑。例如，非编程人员不用考虑搭建环境、数据集成、甚至是策略实盘等问题，只要专注于策略规则编写。

缺点是，平台本质是盈利，免费功能有限，不是数据就是运行时环境的限制，限制较大，难以运行复杂度高或分钟甚至 Tick 级别的策略。要解除限制，就要付费。

还有，公共这些平台都有一个策略保密性的问题。

> 想了解更多的话，推荐阅读 [国内的量化交易平台都有哪些呢？](https://zhuanlan.zhihu.com/p/346393703) 。

## 独立运行于本地环境

另一个方案，基于市面上的开源框架组装集成，有能力的话，可以改造原有框架进行二次开发。

- 优点，解决了诸如安全性、资源限制等问题，一切都自己的掌控中，灵活度高。
- 缺点，要有扎实的编程功底，有一定的开发量，还有数据源和实盘都要单独对接。

国外市面上流行的开源框架的有如 [backtrader](https://www.backtrader.com/)、[backtesting.py](https://kernc.github.io/backtesting.py/)、[vectorbt](https://vectorbt.dev/) 等。

> 早前，quantopian 还没有倒闭时，其开源的 zipline 也是非常流行。

国内有从策略回测到实盘一站式解决方案的 [vnpy](https://www.vnpy.com)，ricequant 开源的适用于国内市场的 [rqalpha](https://github.com/ricequant/rqalpha) 框架等等。

> 上面介绍的都是基于 Python 实现，当然其他语言也有不少优秀框架，这里就不展开了。了解更多，推荐阅读：
> 
> - [6款优秀开源量化策略框架！那款适合你？?](https://zhuanlan.zhihu.com/p/81007132)
> - [当前国内的程序化交易量化交易，有哪些好的框架和工具？](https://www.zhihu.com/question/265096151/answer/2231082866) 
> - [9 Great Tools for Algo Trading](https://hackernoon.com/9-great-tools-for-algo-trading-e0938a6856cd)
> 
> 说明，这些文章里介绍的框架或库不仅仅局限于回测框架。

> 本系列文章基于 Python 从零实现策略回测，故不会借助任何量化框架。如能深入理解，市面上框架的使用将轻而易举。

# 策略回测的步骤

先尝试给策略回测一个完整的定义，如下：

策略回测是是在历史数据的基础上，按依赖预先定义的规则产生信号，模拟交易过程，并对结果评估分析，验证策略的过程。


简单概括为 5 大步骤，如下所示：

```bash
历史数据的获取 --> 策略逻辑的定义 --> 交易信号的生成 --> 交易结果的计算 --> 结果表现的评估
```

其理论基础是，过去的表现在一定程度上上能代表未来。

## 历史数据的获取

回测的第一步就是获取历史数据，俗话说，“兵马未动，粮草先行”。数据获取要考虑两个维度，数据来源和清洗。

数据源有开源数据、从 broker 经纪商或专业的数据供应商。

开源数据，就是通过一些开源数据包获取免费数据，有的是通过爬虫抓取财经网站的数据，有的是个人收集的，类似前面提高的行情录制就是一种采集手段，最后将数据以 HTTP API 的形式提供用户。

> 早前（2018 年）参与过 tushare 的项目，这个包提供了两个大版本，即分别是爬虫形式提供的 [tushare.org](http://tushare.org) 和以数据服务形式提供的 [tushare.pro](https://tushare.pro)，tushare 的作者米哥在金融行业深耕多年，熟悉金融数据整个制作流程，pro 数据可靠性比一般的开源数据质量还是高不少。

通过经纪商 broker 获取数据的方式可通过 **Data API** 或 **行情录制**。**Data API**，部分 broker 会提供获取历史行情数据的 API，但一般有数量限制，只有用于策略初始化，不足以用于策略回测；**行情录制**，直接订阅实时数据保存，这种方式获取的数据最及时，第一手数据，独立维护。但一旦订阅程序出错，数据丢失了就要找其他数据源重新补齐。

> vnpy 提供了 DataRecorder 模块，可用于记录行情数据，支持 tick 和 bar 两种类型数据。查看 [DataRecorder - 实盘行情记录模块](https://www.vnpy.com/docs/cn/data_recorder.html)。

这种方式一般只有获取行情数据，对于技术交易者，已经足够使用了。但金融数据种类丰富，其它如股票选股用的基本面数据、行业分类甚至宏观数据，还需要另外收集，个人不可能完成。

专业的数据服务商，个人难以完成，寻找专业数据服务商是一个快速的解决方案，它们有齐全的数据，种类丰富，甚至还会提供一些特色数据，一个公司专门做这个事情，数据质量自然不必说，部分提供商会提供一些免费数据，不过免费版本限制较多，一般只有日线数据，其他数据通常需要付费。

> 数据源这块，如果展开说，一时也聊不完，推荐几篇文章阅读：
> 
> - [谈一谈量化投资从哪里获取数据](https://zhuanlan.zhihu.com/p/219931158)
> - [如何搭建量化投资研究系统？（数据篇）](https://bigquant.com/wiki/doc/5aac5l2v5pct5bu66yep5yyw5oqv6lwe56cu56m257o757uf77yf77yi5pww5o2u56h77yj-KMtY5btgiN)

数据清洗要处理数据丢失、重复还有异常值。这块内容等讲到数据部分再展开讲。

不得不提一句，高质量的数据对于测试的结果至关重要，所谓 "Garbage in, Garbage out"，垃圾输入必然是垃圾输出，有出乎意料的坏结果。

# 策略规则的定义

量化策略的最基本的组成：进场和出场的规则，此外，实盘的策略还要考虑资金管理与风险控制。

> 量化交易一个核心优势，精确的策略，摒除诸如情绪交易，基于建议或消息交易，让一切没有商量的余地，都在预期之内。

> 同事间聊天，有时会说，如果在跌倒 2700 买入能如何。这种是一个量化策略吗？不是，入场点只是一个交易系统的一部分，而并非全部。文章开头提到，**"10 日与 20 均线金叉入场，死叉出场"**，这是一条量化策略，设计了明确的入场、出场规则。

这部分内容可以说是交易者最关注的核心，是交易真正研究的部分，其他周边都在以它为中心。所谓圣杯，就是想找到一个战无不胜的策略。

一张图来对量化策略做个分类，如下所示：

# 交易信号的生成

# 交易系统的回测

# 结果收益的评估

# 结语

本文重点介绍了为什么要进行策略回测，以及策略回测的步骤，并对每个步骤涉及的要点做了一个粗略介绍。希望给读者在读完本文后，对策略回测所涉及有一个整体蓝图。


下篇文章：[从零开始回测量化交易策略 Part 2: 数据准备篇](../2023-08-12-algo-trading-from-scratch-fetching-data) (待完成)

# 推荐阅读

- [Should Build Your Own Backtester](https://www.quantstart.com/articles/Should-You-Build-Your-Own-Backtester/)
- [The Importance of Backtest Trading Strategies](https://www.investopedia.com/articles/trading/05/030205.asp)
- [Backtesting: How to Backtest, Analysis, Strategy, and More](https://blog.quantinsti.com/backtesting/)
- [Common mistakes to avoid while backtesting to measure results accurately](https://blog.quantinsti.com/common-mistakes-backtesting/)
- [交易策略：什么是回测？为什么要做回测？什么时候不需要回测？...](https://zhuanlan.zhihu.com/p/75909636)
- [什么样的量化交易策略才是最有用的？](https://blog.csdn.net/weixin_42219751/article/details/98177036)

