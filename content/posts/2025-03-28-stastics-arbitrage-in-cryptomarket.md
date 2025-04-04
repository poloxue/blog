---
title: "加密货币市场上的跨品种统计套利"
date: 2025-03-28T14:07:49+08:00
draft: true
comment: true
description: "本文将基于数字货币市场分析统计套利策略。"
---

前几篇套利的文章中，重点聚焦在低风险的套利策略，如期现套利、资金费率套利、跨市场价差等策略。本文将转向一个具有挑战性的领域——统计套利。这个策略不追求无风险收益，而是通过数学建模捕捉资产间的统计规律。

相较于其他套利，统计套利需要一定的数学基础，我也是边学边测试，可能表达上会有不准确之处。

## 什么是统计套利？

统计套利是一种基于数学和概率的量化交易策略，通过识别资产价格之间的统计关系来获利。与传统套利不同，它不依赖无风险定价偏差，而是寻找历史价格存在稳定关联的资产对，当短期价格偏离长期均衡关系时进行反向交易，押注价差回归均值。

举个例子，假设观察到 ETHUSDT 于 BTCUSDT 的价差在过去 6 个月呈现出均值回归特征，当价差偏离其 20 日均值超过 1.5 个标准差时，有约70%的概率会在未来5天内回归至1个标准差范围内。基于这个假设，我们可以构建一个统计套利策略：当价差突破1.5个标准差时建立对冲头寸，待价差回归至1个标准差范围内时平仓。

统计套利的优势在于不受市场方向影响，通过多空对冲保持市场中性。但需注意，其存在模型失效风险，需要严格的风控管理。成功的统计套利往往建立在大量历史数据和稳健的统计检验基础之上。

## 如何分析？

该策略核心在于三个步骤：首先通过相关性分析筛选潜在资产对；其次用协整检验验证长期均衡关系；最后构建价差交易组合。

这里有两个核心的概念：相关性和协整检验。

什么是相关性？

相关性是指两个资产价格变化的联动关系。数值在-1到1之间：1表示完全同涨同跌，-1表示总是反向变化，0表示没有关联。如比特币和以太坊价格经常一起涨跌，它们的相关性就是正数。

为什么要协整检验?

相关性只能说明两个资产同涨同跌的趋势，但无法判断它们价差是否会失控（比如A总是涨得比B多），如 ETH 和 BTC 的价格，经常是同涨同跌，但两者的价差却在不断跨大。

协整性验证的就是验证这个价差是否会在长期保持稳定，具有均值回复特性，而不会无限扩大。

接着就可以做空高估资产，同时做多低估资产，赚到这个价值回归的价差。

## 开始分析

为了简化过程，我先省略从市场去筛选潜在的资产对，用 BTC ETH 作为演示标的。我统计下来，BTC 和 ETH 的配对并不是好的组合。

## 行情数据

我最近想把常用数据的下载函数统一整理到一起，便于后续使用，就创建了一个名为 `findata` 包。

```bash
git clone https://github.com/poloxue/findata
pip install -e .
```

下载行情数据：

```python
from findata import data_ccxt

btc_price = data_ccxt.download("BTC/USDT", start_date="20220101")['close']
eth_price = data_ccxt.download("BTC/USDT", start_date="20220101")['close']
```

## 计算相关性

计算两个时间序列相关性，最常用的是 **Pearson 相关系数**，公式为：

\[
r = \frac{\sum_{i=1}^n (X_i - \bar{X})(Y_i - \bar{Y})}{\sqrt{\sum_{i=1}^n (X_i - \bar{X})^2} \sqrt{\sum_{i=1}^n (Y_i - \bar{Y})^2}}
\]

相关系数解读：
- 结果范围：-1（完全负相关）到 +1（完全正相关）
- 例如：`0.85` 表示强正相关，`-0.12` 表示微弱负相关

Pandas 中提供了 Pearson 相关系数计算函数。

```python
import pandas as pd

correlation = btc_price.corr(eth_price)
print(f"相关系数: {correlation:.3f}")
```

输出：

```bash
相关系数: 0.793
```

可使用价格收益率，而非原始价格以避免伪相关。

```python
btc_returns = btc_price.pct_change()
eth_returns = eth_price.pct_change()

correlation = btc_returns.corr(eth_returns)
print(f"相关系数: {correlation:.3f}")
```

输出：

```bash
相关系数: 0.801
```

也可通过滚动窗口计算，如30天，观察动态相关性变化

```python
correlations = btc_returns.rolling(window=30).corr(eth_returns)
correlations.plot()
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-28-statistic-arbitrage-in-cryptomarket-01.png)

按月推算的相关系数，整体保持正相关，但它的波动范围挺大，从 0.4-0.9，并不稳定。

我可以设置一个相关性的过滤阈值，如 0.9，强相关的情况下再考虑进行后续的 ADFTest 协整性验证。

## ADF 协整验证

假设 BTC ETH 两个资产通过了相关性的初步筛选，现在通过 ADF 检验两个是否存在某种长期均衡关系，即协整验证。

检验协整关系的一种常用方法：

首先用回归分析找出序列之间的关系，然后检验回归结果的残差是否稳定。如果残差稳定，就证明这些序列具有协整关系。

如果是涉及多个变量的复杂情况，也有更高级的检验方法来确定它们之间的关联性。

开始用 Python 实现 ADF 验证过程：

```python
import numpy as np
import statsmodels.api as sm

from statsmodels.tsa.stattools import adfuller

x = btc_price.values
y = eth_price.values

x = sm.add_constant(x)
models = sm.OLS(y, x).fit()
alpha, beta = models.params

residuals = y - (alpha + beta * x[:, 1])
adf_result = adfuller(residuals)

adf_stat, p_value, = adf_result[0], adf_result[1]
```

这段代码得到的有 ADF 统计量 `adf_stat`，`p_value` 和 alpha 和 beta（最小二乘线性回归曲线）。

## 结果绘图；

- scatter图；
- 价差图；

## 策略回测



## 分析工具

开发一个分析工具，从相关性、协整和 ADFTest 分析跨期合约或不同品种之间的相关性，然后开发基于 backtrader 回测配对交易策略，看看效果如何。



