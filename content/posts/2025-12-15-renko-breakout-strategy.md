---
title: "策略1 - Renko ATR 突破"
date: 2025-12-15T05:24:19+08:00
draft: true
comment: true
description: ""
---

本策略是从 Renko 图表类型得到的灵感，一个非常简单的趋势突破策略。核心参数的只有两个，还有三个参数是于仓位管理相关。

## 什么是 Renko？

在说明 renko 是什么之前，先从一个交易中的常见困扰开始。

传统蜡烛图上下翻飞的小影线和反转 K 线常常让人困惑：这到底是正常回调，还是反转的开始？这些由微小波动形成的 "噪音"，是许多错误判断的源头。

是否有办法过滤掉这些噪音呢？

由此就可以引申出一个新的图表类型 - Renko，或者称作 "砖形图"，就是为了这个目标而生的。

这种行情图抛弃了"时间" 的概念，专注于“纯粹的价格变动幅度”。我们需要的是预先设定一个固定的 "砖块大小"（比如股价的10元）。

规则很简单：只有当价格运动超过一个砖块的大小，才会在图上形成一块新的砖；价格变动不足一个砖块时，图表会静止不动。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-13-v1.png)

价格以白涨和黑跌的阶梯状呈现出一种独特的行情图，低于设定阈值的波动都被视为无关紧要的杂波，不予显示。

最终效果是图表因此变得极其清晰，趋势的延续、停滞或反转一目了然，让你能专注于市场的主要运动，不被琐碎的波动干扰判断。

## 砖块大小

Renko 中很重的点是如何确认每个砖块大小，常见的砖块大小策略主要有三种思路。

- 一是采用固定价格数值，例如设定股票每个砖块为0.5元；
- 二是采用动态方法，如 ATR 作为基准，设定砖块大小为一个 ATR ，或固定个数计算，如当前价格/固定个数得到间隔；
- 三是采用百分比模式，例如设定砖块大小为当前资产价格的1%。

ATR 动态计算的方法是最受推崇的方法，因为它能随波动率调整。

## Python 绘制

python 中有 mplfinance 支持绘制 renko 图表，代码如下所示：

```python
import pandas as pd
import mplfinance as mpf

df = pd.read_csv("datas/BTC_4h.csv", index_col="datetime", parse_dates=True)
df = df["2025-10":]

renko_params = {
    "brick_size": 2000,
}

mpf.plot(df, type="renko")

renko_params = {
    "brick_size": "atr",
}
mpf.plot(df, type="renko", renko_params=renko_params)
```

固定砖块大小图表：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-01.png)

ATR 砖块大小：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-02.png)

## 策略设计

基于 Renko 砖块突破的趋势连续性设计一个策略，核心两个参数：连续多个砖块开仓，反转连续多个砖块平仓。Renko 的砖块大小就通过 ATR 计算。

仓位管理逻辑，每单交易的最大损失固定为 2%（24 小时市场），止损单价格在下单价格损失 3 个 ATR，杠杆最大不超过 1.5。

这是一个非常简单的策略。

回测优化：

- 参数范围：开仓突破 1-5 个连续砖块，平仓连续 1-5 个砖块。
- 行情数据：BTC 1小时和 4 小时数据；
- 时间防伪：2020-01-01 到 2025-11-30；

### BTC 的 4 小时上回测。

不同参数走势图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-03.png)

不同评价标准的参数热力图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-04.png)

各方面指标都不错的参数组合（开仓 4，平仓 5）：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-05.png)

所有参数按周再平衡表现：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-06.png)

### BTC 的 1 小时上回测。

不同参数走势图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-08.png)

不同评价标准的参数热力图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-07.png)

各方面指标都不错的参数组合（开仓 5，平仓 4）：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-10.png)

所有参数按周再平衡表现：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-09.png)

总体而言，1 小时上的表现稳定性比 4 小时要好。但无论是 1 小时还是 4 小时，最新一年的表现都比较拉垮。

如果看 4 小时和 1 小时所有参数再平衡年度和月度表现：

4 小时：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-12.png)

1 小时:

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-15-renko-breakout-strategy-11.png)

总结就是今年的行情走的非常令人恶心，太多假突破。
