---
title: "从零量化交易 Part 2: 量化策略篇"
date: 2024-08-06T16:37:03+08:00
draft: true
comment: true
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

本篇为系列文章的第三篇，介绍关于策略的数据准备。

# 推荐阅读

- [Should Build Your Own Backtester](https://www.quantstart.com/articles/Should-You-Build-Your-Own-Backtester/)
- [The Importance of Backtest Trading Strategies](https://www.investopedia.com/articles/trading/05/030205.asp)
- [Backtesting: How to Backtest, Analysis, Strategy, and More](https://blog.quantinsti.com/backtesting/)
- [Common mistakes to avoid while backtesting to measure results accurately](https://blog.quantinsti.com/common-mistakes-backtesting/)

