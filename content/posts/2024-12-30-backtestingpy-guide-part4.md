---
title: "Backtesting.py 教程：由浅入深"
date: 2024-12-12T15:15:18+08:00
draft: true
comment: true
description: "本文继续介绍 **Backtesting.py**，重点是关于它提供的一些交易相关的能力，如做空、止盈止损、订单、资金管理和交易记录等。"
---

本文继续介绍 **Backtesting.py**，重点是关于它提供的一些交易相关的能力，如做空、止盈止损、订单、资金管理和交易记录等。

## 做空、止损和止盈

除了基本的多头交易策略，我们还可以在 Backtesting.py 中实现做空、止损和止盈功能。

### 实现做空
在 RSI 策略的基础上，可以通过以下代码实现做空：
```python
def next(self):
    if crossover(self.daily_RSI, self.upper_bound):
        if self.position.is_long:
            self.position.close()
        self.sell()

    if crossover(self.lower_bound, self.daily_RSI):
        if self.position.is_short:
            self.position.close()
        self.buy()
```
此代码实现了 RSI 超过上限时平多头并开空头，而 RSI 低于下限时平空头并开多头。

### 止损和止盈

通过设置止损（Stop Loss, SL）和止盈（Take Profit, TP），可以更好地控制风险和锁定收益：
```python
def next(self):
    price = self.data.Close[-1]
    sl = 0.95 * price  # 止损设为当前价格的 95%
    tp = 1.15 * price  # 止盈设为当前价格的 115%

    if crossover(self.lower_bound, self.daily_RSI):
        self.buy(sl=sl, tp=tp)

    if crossover(self.daily_RSI, self.upper_bound):
        self.sell(sl=sl, tp=tp)
```
此代码在每次开仓时，同时设置止损和止盈价格。

## 订单大小

在策略中，订单大小的控制至关重要，它直接影响到风险管理和资金利用效率。Backtesting.py 提供了多种设置订单大小的方法，以满足不同交易策略的需求。

### 使用资金比例

可以通过设置 `size` 参数为小数值，指定每笔交易占用可用资金的比例。例如：
```python
self.buy(size=0.1)
```
此代码表示每次买入使用可用资金的 10%。

#### 示例
以下是一个使用资金比例的完整示例：
```python
if crossover(self.lower_bound, self.daily_RSI):
    self.buy(size=0.1)
```
在上述示例中，每次 RSI 低于下限时，将使用当前可用资金的 10% 进行买入。

**注意事项**：
- 默认情况下，Backtesting.py 不支持部分股票买卖，仅支持整数股。因此，交易规模可能会向下取整。

### 使用固定数量

除了使用资金比例，还可以直接指定买入或卖出的数量。例如：
```python
self.buy(size=1)
```
此代码表示每次买入 1 股。

#### 示例
以下是一个按固定数量交易的完整示例：
```python
if crossover(self.lower_bound, self.daily_RSI):
    self.buy(size=1)
```
每次 RSI 低于下限时，将买入 1 股。

### 动态调整订单大小

订单大小可以基于策略动态调整。例如，可以在策略初始化时设置一个变量，并在交易逻辑中引用：
```python
class DynamicSizeStrategy(Strategy):
    def init(self):
        self.position_size = 1

    def next(self):
        if crossover(self.lower_bound, self.daily_RSI):
            self.buy(size=self.position_size)
```
还可以使用优化器来测试不同的订单大小：

```python
bt.optimize(position_size=range(1, 10))
```

### 与其他条件结合

订单大小可以与其他条件结合，例如动态计算当前价格低于指定阈值的天数：
```python
if bars_since(self.daily_RSI > self.upper_bound) >= 5:
    self.buy(size=2)
```
此逻辑表示如果 RSI 超过上限已经过去了至少 5 天，则买入 2 股。

### 应用场景

- **资金管理**：通过比例买入控制资金利用率，降低单笔交易的风险。
- **网格交易**：在特定价格区间内分批买入或卖出。
- **优化策略**：通过优化订单大小，找到最优交易规模。

通过灵活设置订单大小，可以更好地满足策略的需求，同时提高资金的利用效率和交易的灵活性。

## 交易记录导出

在策略回测完成后，我们通常希望能够导出交易记录，便于进一步分析或保存。

### 提取交易记录

Backtesting.py 会在回测结果中生成一个包含所有交易详情的 Pandas 数据框，可以通过以下方式提取：
```python
trades = stats['_trades']
print(trades.to_string())
```

### 交易记录内容

交易记录包括以下信息：

- **Entry/Exit**：交易的开仓和平仓时间。
- **Size**：交易的头寸大小。
- **P/L**：交易的利润或亏损。
- **Duration**：交易的持续时间。

这些数据可以方便地用于后续分析，例如计算平均持仓时间、每笔交易的盈亏比等。

### 导出到文件

如果需要保存交易记录，可以将其导出为 CSV 文件：

```python
trades.to_csv('trades.csv')
```
或者保存为 Excel 文件：
```python
trades.to_excel('trades.xlsx')
```

### 可视化交易记录

导出的交易记录可以使用工具（如 Excel 或 Python 的可视化库）进行分析。例如，使用 Matplotlib 绘制每笔交易的盈亏分布：
```python
import matplotlib.pyplot as plt

trades['P/L'].hist(bins=20)
plt.title('Profit/Loss Distribution')
plt.xlabel('Profit/Loss')
plt.ylabel('Frequency')
plt.show()
```
此代码生成一个柱状图，展示交易盈亏的分布情况。

通过灵活导出和处理交易记录，可以更全面地分析策略表现并进行优化。


