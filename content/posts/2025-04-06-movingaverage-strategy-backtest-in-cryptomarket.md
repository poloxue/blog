---
title: "策略回测：加密货币市场上双均线策略"
date: 2025-04-06T17:59:11+08:00
draft: true
comment: true
description: ""
---

本文将回测双均线交叉策略在加密货币市场中的表现。作为一种简单且经典的趋势跟随策略，双均线交叉易于理解和实现。

本文将从最基础的交易规则出发，逐步调整并引入参数优化与风险控制，层层展开，并提供完整的回测分析结果。

均线先采用 SMA 简单移动均线，后续引入 EMA（指数移动均线）、WMA（加权移动均线） 和 AMA（考夫曼自适应均线）这三个均线类型。

## 交易规则

首先，基于简单移动均线实现这个最简单的双均线策略，定义其入场与出场规则。

**仓位管理：**

仓位管理先采用 **满仓入场**，每次交易时将所有资金投入当前市场。简单直接，但在波动较大时可能导致较高风险，后面考虑加入更灵活的仓位管理策略。

**入场条件：**

做多：当快线（短期均线）上穿慢线（长期均线）时，视为上涨信号，建立多头仓位。

做空：当快线下穿慢线时，视为下跌信号，建立空头仓位。

**出场条件：**

平多：若已有多头仓位，当快线再次下穿慢线，视为趋势反转信号，平掉多头仓位。

平空：若已有空头仓位，当快线再次上穿慢线，视为止盈或止损信号，平掉空头仓位。

## 初步回测

**数据下载**：使用币安的 BTC/USDT 交易对，获取自 2020 年 1 月 1 日至 2025 年 4 月 5 日的日线行情数据。

**参数定义**：快均线（短期均线）设置为 10，慢均线（长期均线）设置为 20。

### 回测净值曲线：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-06-movingaverage-strategy-backtest-in-cryptomarket-01.png)

通过回测，默认参数下的策略最终显示收益率为 -71%（即净值从 1 降至 0.29）。这一策略表现明显远低于预期，亏损幅度较大。

得到了一个不理想的回测结果，但这正是优化和调整策略的起点。接下来，我们将进行参数优化，优化策略，探索更多可能，进一步改进策略表现。

## 参数优化

上述回测是基于固定的快慢均线周期（分别为 10 和 20），且使用的是固定的日线数据。我们将这些参数调整为可优化的变量，尝试通过优化过程找出最适合的组合。

回测仍然以日线行情为基础，网络遍历优化参数如下：

```python
short_periods = range(5, 30, 5)
long_periods = range(20, 100, 5)
```

按 SQN 选择最优参数组合为`short_period=25` 和 `long_period=90`，净值曲线如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-06-movingaverage-strategy-backtest-in-cryptomarket-03.png)

常见的评价指标如下所示：

```
最终净值: 2315120.58
最终收益率：132%
夏普比率: 0.529
最大回撤: 81.8%
```

回撤有足足 81.8%，还是相当大的。

也可以看看最佳收益率净值曲线：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-06-movingaverage-strategy-backtest-in-cryptomarket-02.png)

从净值曲线上看，长期的有效性太差。

## 行情周期参数

加入行情周期参数，网格遍历优化参数：

```python
intervals = ["30m", "1h", "2h", "4h", "12h", "1d"]
```

常见评价指标如下所示：

## 风险控制

当前策略回测较大，为提升策略的风险管理能力，为其加上风险控制逻辑。

### 设置止损

在当前入场位置回撤 3 倍 ATR 即止损，倍数可设置为可配置参数。

假设 ATR 为 3 时，回测的净值曲线如下：

常见评价指标：

### 最大损失百分比

前面虽已配置止损，但如果波动过大，止损金额也会太大，为防止类情况，配置最大损失不超过仓位的 2%。

控制最大损失后的回测净值曲线：

评价指标：

## 回测汇总

评价指标对比表格：

指标           | v1 策略   | v2 版策略  | v3 版策略 
-------------- | --------- | ---------- | -------------
收益           |           |            | 
最大回测       |           |            |
夏普比率       |           |            | 
年化收益       |           |            | 
年华波动率     |           |            | 

年度表现对比表格：

## 总结

本文的完整策略回测代码可关注公众号回复 cryptomovingaverage 获取代码地址。
