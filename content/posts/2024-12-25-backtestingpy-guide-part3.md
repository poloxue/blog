---
title: "Backtesting.py 教程：由浅入深"
date: 2024-12-12T15:15:18+08:00
draft: true
comment: true
description: "本文将介绍 **Backtesting.py**，一个轻量级的 Python 的交易回测框架。"
---

## 多时间框架策略

在某些策略中，我们希望结合多时间框架的数据来提高交易决策的可靠性。例如，除了每日 RSI（Relative Strength Index），我们还可以添加每周 RSI，并要求在两个时间框架的指标满足条件时才进行交易。

### 实现多时间框架策略

以下是如何在策略中实现多时间框架的步骤：

1. **导入所需函数**：
```python
from backtesting.lib import resample_apply
```

2. **定义每周 RSI**：

在策略的 `__init__` 方法中添加每周 RSI 的计算：
```python
self.daily_RSI = self.I(TA.RSI, self.data.Close, self.rsi_window)
self.weekly_RSI = resample_apply('W', TA.RSI, self.data.Close, self.rsi_window)
```
这里使用 `resample_apply` 函数，将每日数据降采样为每周数据，并计算 RSI。

3. **修改交易逻辑**：

在 `next` 方法中，结合每日和每周 RSI 来决定交易：

```python
if crossover(self.daily_RSI, self.upper_bound) and self.weekly_RSI[-1] > self.upper_bound:
    self.position.close()

if crossover(self.lower_bound, self.daily_RSI) and self.weekly_RSI[-1] < self.lower_bound:
    self.buy()
```

此逻辑要求每日 RSI 与阈值交叉，并且每周 RSI 同时满足条件。

### 代码示例
以下是完整的多时间框架策略示例：
```python
from backtesting.lib import resample_apply

class MultiTimeFrameRSIStrategy(Strategy):
    rsi_window = 14
    upper_bound = 70
    lower_bound = 30

    def init(self):
        self.daily_RSI = self.I(TA.RSI, self.data.Close, self.rsi_window)
        self.weekly_RSI = resample_apply('W', TA.RSI, self.data.Close, self.rsi_window)

    def next(self):
        if crossover(self.daily_RSI, self.upper_bound) and self.weekly_RSI[-1] > self.upper_bound:
            self.position.close()

        if crossover(self.lower_bound, self.daily_RSI) and self.weekly_RSI[-1] < self.lower_bound:
            self.buy()
```

### 策略优化与扩展

- **参数优化**：
  可以将 `weekly_RSI` 的窗口期设为一个独立参数，并使用优化器来测试最佳值。

- **更多条件**：
  添加更多条件，例如检查每日 RSI 是否持续下降，以增强策略的灵活性。

通过结合多时间框架数据，我们可以更好地过滤噪声，提高交易决策的准确性。

