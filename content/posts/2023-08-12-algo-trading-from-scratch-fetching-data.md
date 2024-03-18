---
title: "从零量化交易 Part 2: 数据准备篇"
date: 2024-08-06T16:37:03+08:00
draft: true
comment: true
toc: "floating"
---

> 本系列文章目标是，基于 Python 从零实现策略回测，示例策略选用最简单的双均线策略。
>
> - [从零开始量化交易 Part 1: 基本介绍篇](../2023-08-06-algo-trading-from-scratch-introduction)
> - [从零开始量化交易 Part 2: 数据准备篇](../2023-08-12-algo-trading-from-scratch-fetching-data) (待完成)
> - [从零开始量化交易 Part 3: 量化策略篇](../2023-08-16-algo-trading-from-scratch-strategy-rules) (待完成)
> - [从零开始量化交易 Part 4: 交易信号篇](../2023-08-06-algo-trading-from-scratch-strategy-signals) (待完成)
> - [从零开始量化交易 Part 5: 策略回测篇](../2023-08-06-algo-trading-from-scratch-backtesting-strategy) (待完成)
> - [从零开始量化交易 Part 6: 收益评估篇](../2023-08-06-algo-trading-from-scratch-evaulating-returns) (待完成)
> - [从零开始量化交易 Part 7: 风险管理篇](../2023-08-06-algo-trading-from-scratch-risk-management) (待完成)
> - [从零开始量化交易 Part 8: 实盘接入篇](../2023-08-06-algo-trading-from-scratch-live-trading) (待完成)

本篇为系列文章的第一篇，介绍关于量化交易的数据部分。数据获取要考虑两个维度，数据来源和清洗。

# 数据源

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


# 推荐阅读

- [Should Build Your Own Backtester](https://www.quantstart.com/articles/Should-You-Build-Your-Own-Backtester/)
- [The Importance of Backtest Trading Strategies](https://www.investopedia.com/articles/trading/05/030205.asp)
- [Backtesting: How to Backtest, Analysis, Strategy, and More](https://blog.quantinsti.com/backtesting/)
- [Common mistakes to avoid while backtesting to measure results accurately](https://blog.quantinsti.com/common-mistakes-backtesting/)

