---
title: "由浅入深策略回测 Part 1: 基本介绍"
date: 2023-08-06T16:37:03+08:00
draft: true
comment: true
---

本系列文章的目标是，基于 Python 从零实现回测策略。本篇为系列文章的第一篇，介绍关于策略回测的基础内容。

# 什么是策略回测？

通过观察，我发现了一个可能赚钱的策略，10 日与 20 均线金叉入场，死叉出场。现在如果我在实盘中使用这个策略，它能产生好结果呢？如果不是过于自信，绝不敢轻易下这个结论。实际上，交易时经常会闪现各种想法，觉得好像发现了诀窍，但基本都经历不了测试的考验，以亏损收尾。

交易市场中，有不少人依赖没有验证过的经验、他人的建议，或主观的情绪交易，而非一套经过历史验证的交易系统。上面所说的测试即策略回测，通过把想法在历史数据上评估和分析，得出这个策略的好坏结论。

策略回测是在历史数据的基础上，按依赖预先定义的规则产生信号，模拟交易过程，并对结果评估分析，验证策略的过程。其理论基础是，过去的表现在一定程度上上能代表未来。

在明白策略回测重要性后，究竟如何开始回测？在开始原始手撸，介绍下一般情况下的策略回测的途径吧。

# 常见的回测途径

策略回测的途径，从易到难，即可基于现成的量化平台，也可基于开源框架组装集成独立运行于本地环境。

## 基于专业的量化平台

现在国内有不少专业的量化，即有支持非编程回测，也有通过编程实现的平台。有一些平台有自己的特色。从核心功能上评价，它们帮用户解决了上手难的问题，体验丝滑。例如，非编程人员不用考虑搭建环境、数据集成、甚至是策略实盘等问题，只要专注于策略规则编写。

缺点是，平台本质是盈利，免费功能有限，不是数据就是运行时环境的限制，限制较大，难以运行复杂度高或分钟甚至 Tick 级别的策略。要解除限制，要选择付费版。另外，使用这些平台，还有一个策略保密性的问题。

> 想了解更多的话，推荐阅读 [国内的量化交易平台都有哪些呢？](https://zhuanlan.zhihu.com/p/346393703) 。

## 独立运行于本地环境

解决了诸如安全性、资源限制等问题，基于市面上的开源框架或独立开发，组装实现要求功能，如果有扎实的编程功底，一切都自己的掌控中，灵活度高。相应的缺点是开发量大且复杂，而且数据源问题要单独解决，实盘也要独立对接。

市面上开源框架的有如 [backtrader](https://www.backtrader.com/)、[backtesting.py](https://kernc.github.io/backtesting.py/)、[vectorbt](https://vectorbt.dev/) 等，国内也有从策略回测到实盘一站式解决方案的 [vnpy](https://www.vnpy.com)，以及 ricequant 开源的适用于国内市场的 [rqalpha](https://github.com/ricequant/rqalpha) 框架等等。早前，quantopian 还没有倒闭时，其开源的 zipline 也是非常流行。

> 这里介绍的都是基于 Python 实现，当然其他语言也有不少优秀框架，这里就不展开了。了解更多，推荐阅读：
> 
> - [6款优秀开源量化策略框架！那款适合你？?](https://zhuanlan.zhihu.com/p/81007132)
> - [当前国内的程序化交易量化交易，有哪些好的框架和工具？](https://www.zhihu.com/question/265096151/answer/2231082866) 
> - [9 Great Tools for Algo Trading](https://hackernoon.com/9-great-tools-for-algo-trading-e0938a6856cd)
> 
> 说明，这些文章里介绍的框架或库不仅仅局限于回测框架。


> 本系列文章基于 Python 从零实现策略回测，故不会借助任何量化框架。如能深入理解，市面上框架的使用将轻而易举。

# 策略回测的步骤

回顾对策略回测的描述：策略回测是在历史数据的基础上，按依赖预先定义的规则产生信号，模拟交易过程，并对结果评估分析，验证策略的过程。

从中得到策略回测的 5 大步骤：

```bash
历史数据的获取 --> 策略规则的定义 --> 交易信号的生成 --> 交易结果的计算 --> 结果表现的评估
```

## 历史数据

第一步就是获取历史数据。历史数据的来源一般有两个分类，即数据源头 broker 经纪商和专业的数据供应商。

通过经纪商 broker 获取数据，有两种方式：

**Data API**，部分 broker 会提供获取历史行情数据的 API，但一般有数量限制，只有用于策略的初始化，并不足以用于策略回测；

**行情录制**，

不得不提一句，高质量的数据对于测试的结果至关重要，有一句名言是 "Garbage in, Garbage out"，将会有出乎意料的坏结果。

# 推荐阅读

- [Should Build Your Own Backtester](https://www.quantstart.com/articles/Should-You-Build-Your-Own-Backtester/)
- [The Importance of Backtest Trading Strategies](https://www.investopedia.com/articles/trading/05/030205.asp)
- [Backtesting: How to Backtest, Analysis, Strategy, and More](https://blog.quantinsti.com/backtesting/)
- [Common mistakes to avoid while backtesting to measure results accurately](https://blog.quantinsti.com/common-mistakes-backtesting/)

