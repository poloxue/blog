---
title: "Backtesting.py 实现做空、止损、止盈、订单和交易记录"
date: 2024-12-28T15:15:18+08:00
draft: true
comment: true
description: "本文继续介绍 **Backtesting.py**，重点是关于它提供的一些交易相关的能力，如做空、止盈止损、订单和交易记录等。"
---

本文继续探索 **Backtesting.py** 的使用，重点介绍它提供的交易方法，包括如何做空、止盈止损、订单和交易记录等。

## 做空

之前的文章只演示了 `SMACrossStrategy` 的多头交易，**Backtesting.py** 实际也支持空头策略，实现起来也很简单。

示例：

```python
import talib

class SMACrossStrategy(Strategy):
    def init(self):
        self.fast_ma = self.data.Close.rolling(window=50).mean()
        self.slow_sma = self.data.Close.rolling(window=200).mean()

    def next(self):
        if self.fast_sma[-1] > self.slow_sma[-1]:
            if self.position.is_short:
                self.position.close()  # 平空头
            self.buy()

        elif self.short_sma[-1] < self.long_sma[-1]:
            if self.position.is_long:
                self.position.close()  # 平多头
            self.sell()
```

## 止损和止盈

为了更好地管理风险，我们可以在每次开仓时同时设置止损和止盈。以下是如何在策略中添加止损和止盈的代码：

```python
class SMACrossStrategy(Strategy):
    def init(self):
        # 定义短期与长期均线
        self.short_sma = self.data.Close.rolling(window=50).mean()
        self.long_sma = self.data.Close.rolling(window=200).mean()

    def next(self):
        price = self.data.Close[-1]
        sl = 0.95 * price  # 止损设为当前价格的 95%
        tp = 1.15 * price  # 止盈设为当前价格的 115%

        # 短期均线上穿长期均线，开多头并设置止损止盈
        if self.short_sma[-1] > self.long_sma[-1]:
            if self.position.is_short:
                self.position.close()  # 平空头
            self.buy(sl=sl, tp=tp)

        # 短期均线下穿长期均线，开空头并设置止损止盈
        elif self.short_sma[-1] < self.long_sma[-1]:
            if self.position.is_long:
                self.position.close()  # 平多头
            self.sell(sl=sl, tp=tp)
```

在这个示例中，我们通过计算当前价格的止损和止盈位置（分别为当前价格的 95% 和 115%），并在每次开仓时设置止损和止盈。当市场价格触及这些点时，交易将自动平仓。

## 订单大小

在策略中，订单大小的控制非常关键，它直接影响到交易的风险管理与资金利用效率。Backtesting.py 提供了多种设置订单大小的方法，以下是几种常见的设置方式。

### 使用资金比例

通过设置 `size` 参数为小数值，可以指定每笔交易占用可用资金的比例。例如：

```python
self.buy(size=0.1)
```

此代码表示每次买入时，使用可用资金的 10%。

以下是一个基于资金比例的完整示例：

```python
class SMACrossStrategy(Strategy):
    def init(self):
        self.short_sma = self.data.Close.rolling(window=50).mean()
        self.long_sma = self.data.Close.rolling(window=200).mean()

    def next(self):
        if self.short_sma[-1] > self.long_sma[-1]:
            self.buy(size=0.1)  # 使用资金的10%买入
        elif self.short_sma[-1] < self.long_sma[-1]:
            self.sell(size=0.1)  # 使用资金的10%卖出
```

### 使用固定数量

除了资金比例，你还可以直接指定每次交易的数量：

```python
self.buy(size=1)
```

此代码表示每次买入 1 股。

以下是一个按固定数量交易的完整示例：

```python
class SMACrossStrategy(Strategy):
    def init(self):
        self.short_sma = self.data.Close.rolling(window=50).mean()
        self.long_sma = self.data.Close.rolling(window=200).mean()

    def next(self):
        if self.short_sma[-1] > self.long_sma[-1]:
            self.buy(size=1)  # 每次买入1股
        elif self.short_sma[-1] < self.long_sma[-1]:
            self.sell(size=1)  # 每次卖出1股
```

### 动态调整订单大小

在一些情况下，我们希望根据市场情况动态调整订单大小。可以在策略初始化时设置一个变量，并在交易逻辑中引用：

```python
class DynamicSizeStrategy(Strategy):
    def init(self):
        self.position_size = 1  # 初始化订单大小

    def next(self):
        if self.short_sma[-1] > self.long_sma[-1]:
            self.buy(size=self.position_size)
        elif self.short_sma[-1] < self.long_sma[-1]:
            self.sell(size=self.position_size)
```

还可以使用优化器来测试不同的订单大小：

```python
bt.optimize(position_size=range(1, 10))
```

### 与其他条件结合

订单大小也可以根据其他条件进行调整。例如，可以在价格突破某个重要水平时调整订单大小：

```python
class DynamicSizeStrategy(Strategy):
    def init(self):
        self.position_size = 1  # 初始化订单大小

    def next(self):
        if self.short_sma[-1] > self.long_sma[-1] and self.data.Close[-1] > 100:
            self.buy(size=2)  # 当价格大于100时，买入2股
        elif self.short_sma[-1] < self.long_sma[-1]:
            self.sell(size=1)  # 否则，买入1股
```

### 应用场景

- **资金管理**：通过资金比例控制每笔交易的规模，有效降低单笔交易的风险。
- **网格交易**：在特定的价格区间内分批买入或卖出。
- **策略优化**：通过优化订单大小，找到最适合的交易规模。

## 交易记录导出

回测完成后，我们通常希望能够导出交易记录，便于进一步分析或保存。Backtesting.py 提供了简单的接口来提取和保存交易记录。

### 提取交易记录

Backtesting.py 会在回测结果中生成一个包含所有交易的详细数据。可以通过以下代码提取并查看这些记录：

```python
trades = stats['_trades']
print(trades.to_string())
```

### 交易记录内容

交易记录包括以下信息：

- **Entry/Exit**：交易的开仓和平仓时间。
- **Size**：交易的头寸大小。
- **P/L**：交易的盈亏。
- **Duration**：交易的持续时间。

### 导出到文件

如果需要保存交易记录，可以将其导出为 CSV 或 Excel 文件：

```python
trades.to_csv('trades.csv')
```

或保存为 Excel 文件：

```python
trades.to_excel('trades.xlsx')
```

### 可视化交易记录

导出的交易记录可以使用工具（如 Excel 或 Python 的可视化库）进行分析。以下是一个简单的使用 Matplotlib 可视化交易盈亏的示例：

```python
import matplotlib.pyplot as plt

trades['P/L'].hist(bins=20)
plt.title('Profit/Loss Distribution')
plt.xlabel('Profit/Loss')
plt.ylabel('Frequency')
plt.show()
```

这段代码将展示一个柱状图，显示每笔交易的盈亏分布。

---

通过 Backtesting.py，我们可以轻松地实现做空、止损止盈、订单管理以及资金管理等功能，为量化交易策略的开发提供强大的支持。灵活的订单管理和交易记录导出功能，能够帮助我们更好地优化策略，提升交易效率。

