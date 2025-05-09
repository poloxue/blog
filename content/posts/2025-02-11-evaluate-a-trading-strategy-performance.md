---
title: "如何评价交易策略的表现？"
date: 2025-02-11T15:17:49+08:00
draft: false
comment: true
description: "在设计一个量化策略时，常会面临这样的疑问：我的策略好不好？怎么判断它是否真正有效？"
---

在设计一个量化策略时，常会面临这样的疑问：我的策略好不好？怎么判断它是否真正有效？

本文将尝试从这个角度触发，思考如何判断一个策略的好坏。

## 概要引言

前面写了几篇博文，介绍如何用 backtesting.py 开发策略，当一个策略回测完成后会得到如下的信息。

```bash
Start                   2004-08-19 00:00:00  # 回测开始时间
End                     2013-03-01 00:00:00  # 回测结束时间
Duration                 3116 days 00:00:00  # 回测总时长
Exposure Time [%]                 61.545624  # 持仓暴露时间百分比（处于市场中的时间比率）
Equity Final [$]                 99485.0574  # 回测结束时的账户净值
Equity Peak [$]                 100607.2574  # 回测期间账户达到的最高净值
Return [%]                       894.850574  # 总收益率（百分比）
Buy & Hold Return [%]            703.458242  # 买入并持有策略的收益率（百分比），用于对比
Return (Ann.) [%]                 30.934891  # 年化收益率（百分比）
Volatility (Ann.) [%]             32.215003  # 年化波动率（百分比），衡量收益波动性
Sharpe Ratio                       0.960263  # 夏普比率（风险调整收益率）
Sortino Ratio                      2.125336  # 索提诺比率（只考虑下行风险的风险调整收益率）
Calmar Ratio                       1.497421  # 卡玛比率（年化收益率与最大回撤的比值）
Max. Drawdown [%]                -20.658779  # 最大回撤（百分比），历史上账户净值的最大跌幅
Avg. Drawdown [%]                 -3.678412  # 平均回撤（百分比），所有回撤期间的平均亏损
Max. Drawdown Duration    584 days 00:00:00  # 最大回撤持续时间（最长亏损期）
Avg. Drawdown Duration     38 days 00:00:00  # 平均回撤持续时间
# Trades                                 48  # 总交易次数
Win Rate [%]                      64.583333  # 胜率（百分比），盈利交易的比率
Best Trade [%]                     57.11931  # 最佳交易收益率（百分比）
Worst Trade [%]                  -12.446769  # 最差交易收益率（百分比）
Avg. Trade [%]                     4.923007  # 平均每笔交易的收益率（百分比）
Max. Trade Duration       121 days 00:00:00  # 最长单笔交易持续时间
Avg. Trade Duration        39 days 00:00:00  # 平均单笔交易持续时间
Profit Factor                      4.729158  # 盈亏比（盈利总额与亏损总额的比值）
Expectancy [%]                      5.66852  # 期望收益率（百分比），长期平均每笔交易收益
SQN                                2.643361  # 系统质量数（衡量交易系统稳定性的指标）
```

上面有很多指标，具体都是什么含义呢？

评价一个策略要包含多个角度，不同的角度有对应指标衡量策略相应的表现。

如果你比较在意收益，就得关注那些能直观反映盈利情况的指标，比如总收益率、年化收益率等。

如果你更担心风险，那就得看看可能的最大亏损，比如最大回撤，了解在最糟糕情况下你的资金可能承受多少损失。

一个策略的好坏往往不仅仅取决于收益或风险单一方面，还可以将两者同时考虑进来。

除了收益与风险，其他维度，如交易频率，交易太频繁可能会增加成本和操作难度，而交易频率过低又可能错失良机。

对于短线交易，胜率和盈亏比也是一个不可忽略的因素

## 盈利能力

衡量策略盈利能力最直接的方式就是查看 **总收益率**。但仅仅看总收益可能并不全面，你还需要考虑到收益的**稳定性**。

**年化收益率**：年化收益率能帮助你更好地评估长期的盈利能力，它不仅考虑了总收益，还考虑了时间因素。如果年化收益率过低，意味着策略的盈利潜力有限。

上面策略的总收益为 894.85%。从数据来看，单纯的回报看起来非常好。但年化收益率（**Return (Ann.) [%]**）为30.93%，这表示你每年平均获得30.93%的收益，这样的年化收益在市场中算是相当不错的。


## 风险评估

交易策略不仅要追求盈利，更重要的有效控制风险。我经常听到一句话，成功的交易一定能很好的管理风险的。**最大回撤**和**波动率** 就是两个量化策略风险的指标。

**最大回撤**：最大回撤是指策略在某一周期内，账户从最高点到最低点的最大亏损幅度。

**波动率**：波动率用于衡量收益的波动程度。如果波动率过大，意味着策略在盈利的同时可能也会伴随较大的亏损风险。

在上面的回测报告中，策略的最大回撤为 **-20.66%**。这意味着，尽管策略有不错的回报，但在某些市场情况下，策略的资金也曾经历过接近21%的亏损。年化波动率为 **32.22%**，表明该策略在年度内的波动幅度较大，但考虑到它有着 **30.93%** 的年化收益率，也不算太差。

## 风险与收益

单纯的盈利数据可能掩盖了潜在的风险，因此，评估策略时需要加入**风险调整后的收益**指标，如**夏普比率**和**索提诺比率**。

**夏普比率**（Sharpe Ratio）：夏普比率用来衡量单位风险带来的超额收益，公式为：
$$
\text{Sharpe Ratio} = \frac{\text{Annualized Return} - \text{Risk-Free Rate}}{\text{Annualized Volatility}}
$$

在这个回测中，夏普比率为0.96，虽然不算很高，但也说明策略在风险控制上做得较为稳定。

**索提诺比率**（Sortino Ratio）：与夏普比率类似，但索提诺比率只关注下行风险，这适合那些特别注重防止亏损的交易者。毕竟，对交易而言，下行风险才是风险。

其公式为：
$$
\text{Sortino Ratio} = \frac{\text{Annualized Return} - \text{Risk-Free Rate}}{\text{Downside Deviation}}
$$

回测报告中，索提诺比率为2.13，表示策略在承担下行风险时，能获得相对较高的收益。

## 胜率和盈亏比

如果你是做短线交易，**胜率**和**盈亏比**是非常重要的指标。

**胜率**：胜率是你在所有交易中获胜的比率。
  
**盈亏比**（Profit Factor）：是你每次盈利的金额与亏损的金额之比。

在这个回测案例中，策略的胜率为**64.58%**，表明超过一半的交易是成功的，表现不错。然而，胜率的高低不能单独作为好坏的标准，还需要结合盈亏比来评估。**盈亏比** 为4.73，说明每赚 4.73 美元，亏损大约是 1 美元，表现非常良好。盈亏比高说明策略的风险回报比良好，即便胜率不算非常高，仍然可以盈利。

## 交易频率

交易频率反映了你策略的活跃度，不同的市场环境可能需要不同的操作节奏。你可以根据策略的性质来判断交易频率是否合适。

**高频交易策略**：如果你的策略是基于短期波动的快速决策，那么频繁的交易可能是必需的。但这种策略对市场的流动性要求较高，风险也较大。
  
**低频交易策略**：比如趋势跟随或者长期持仓策略，它们通常更加稳定，但可能错过一些短期的机会。

我测试的这个策略是趋势最总策略，3000 多天的 **交易总次数** 为 48 次，频率不高，说明这是一个相对低频的交易策略。

## 综合评价指标

前面讲了评价策略的各个维度的指标，是否还记得之前的 backtesting.py 优化入门那篇文章，里面还有一个评价指标，SQN。

SQN 是衡量交易策略稳健性的一个综合指标，其计算公式为：

$$
SQN = (平均每笔交易收益 × √交易次数) / 每笔交易收益的标准差
$$

一句话解释它的含义：SQN 通过结合平均收益（越高越好）、收益波动性（越低越好）以及交易次数（越多越好），来衡量策略的稳健性和质量，值越高代表策略表现越优。从指标的公式可以看出，这个指标即考虑收益还考虑风险，同时包括交易的次数。

如上的回测报告中的 SQN 值为 2.643361，说明策略在收益、风险和交易频率这几个维度上已经取得了较好的平衡。

## 回测与实盘

在补充几句关于回测与实盘的看法。

回测能提供一个策略框架，但真正的考验是策略在实盘中的表现。可能回测结果好，但实际操作中却无法复刻那样的效果。

**回测的好处**：通过历史数据的回测，你可以初步了解策略的优缺点、潜在的收益和风险。

**实盘的挑战**：真实的市场中，你可能会面临滑点、手续费、情绪等额外的因素，这些都可能影响策略的表现。

回测结果只是一个参考，但并不是说它不重要，如果连回测都无法做到好的表现，实盘表现出色的概率基本为无。

---

## 总结

评价交易策略的表现并不复杂，只需要从盈利能力、风险管理、胜率和盈亏比等几个核心指标入手。在实际操作中，可能还要适时调整策略的交易频率、控制风险，。

希望本文能让你更清晰地评价自己的交易策略，帮助自己做出更好的决策，避免过度看中收益，而忽略其他评价维度。