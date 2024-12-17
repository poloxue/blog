---
title: "Backtesting.py 教程：由浅入深"
date: 2024-12-12T15:15:18+08:00
draft: true
comment: true
description: "本文将介绍 **Backtesting.py**，一个轻量级的 Python 的交易回测框架。"
---

在算法交易中，验证一个策略是否有效至关重要，但该如何验证呢？

最快捷的方式就是进行交易回测（Backtesting） ，即通过历史数据，模拟策略在过去市场中的表现，帮助我们评估收益和风险，从而避免盲目实盘操作。

那该如何回测呢？

回测的方式有很多，既可以手工测试、也可以借助 excel 或是 python 的向量计算库，如 numpy 和 pandas 进行。不过这些方式都比较原始，我们可直接借助市面一些现成的回测框架，把重点放在策略的逻辑上。

本文将先介绍 **Backtesting.py**，一个 Python 实现轻量级、事件驱动的回测框架。之所以先介绍它，主要是它非常简单。

## 什么是 Backtesting.py

Backtesting.py 是一个轻量级的 Python 回测框架，专注于策略回测的核心功能，适合快速实现和测试交易策略。与其他回测框架，如 VectorBT 或 Backtrader 等回测框架相比，Backtesting.py 简化了使用流程，没有过多复杂功能，自然地，它更加简单易用与高效。

Backtesting.py 内置了策略参数优化工具，可与 Pandas 数据框和 NumPy 数组兼容，易于集成。当然简单是有代价的，它的缺点是不支持多资产交易和分数股交易，这也限制了它的普适性。

## 快速开始

让我们快速上手 Backtesting.py 的使用吧。

### 安装

确保你已经有了 Python 环境，使用如下命令安装 backtesting.py 和 TA-Lib 两个依赖包。

```bash
pip install backtesting TA-Lib
```

如果 TA-Lib 安装遇到困难，请查看 [TA-Lib 安装指南](https://ta-lib.github.io/ta-lib-python/install.html)，或者也可以选择其他技术指标库，如 pandas-ta。

### 编写第一个策略

开始通过 backtesting.py 实现第一个策略，就以均线交叉策略为例。策略的规则也很简单，就是 SMA 金叉买入、死叉卖出。

**导入必要的模块**

首先，创建一个名为 `strategy.py` 的 Python 文件，开始编写我们的策略。

导入要用到的模块：

```python
from backtesting import Backtest, Strategy
from backtesting.test import GOOG  # 示例数据
import talib
```

简单起见，我们就用 backtestingpy 提供的样例数据 GOOG，即 Google 某段时间点的价格数据来演示。

**定义策略类**

我们定义一个名为 `MovingAverageCrossStrategy` 策略类，继承 `Strategy`：

```python
class MovingAverageCrossStrategy(Strategy):
  # 参数
  fast_ma_window = 10
  slow_ma_window = 20

  def init(self):
    self.fast_ma = self.I(talib.SMA, self.data.Close, timeperiod=self.fast_ma_window)
    self.slow_ma = self.I(talib.SMA, self.data.Close, timeperiod=self.slow_ma_window)

  def next(self):
    if self.fast_ma[-1] > self.slow_ma[-1] and self.position.size == 0:
      self.buy()
    elif self.fast_ma[-1] < self.slow_ma[-1] and self.position.size > 0:
      self.position.close()
```

在 Backtestingpy 中实现的策略要继承 `backtesting` 下 `Strategy` 类，实现它的两个方法：`init` 和 `next`。`Strategy` 是框架提供的基类，它规定了策略结构和生命周期，init 方法用于初始化，如指标的向量计算，`next` 中用于实现提供数据的每个 bar 的执行逻辑。

策略类上的两个变量 `fast_ma_window` 和 `slow_ma_window` 是我们定义的策略参数，代表了均线的快线和慢线的周期，默认设置为 10 和 20。它们是可配置的，在调用策略时，是可以修改这两个参数的值的。

在 `init` 初始化阶段，我们可以通过 talib 计算 SMA 均线的快慢线。数据可以通过 `self.data` 访问，指标计算用到了收盘价，即 `self.data.Close`。

在 `next` 交易逻辑阶段，只需要拿到当前的指标数据，判断是否要入场就行了，`self.fast_ma[-1]` 和 `self.slow_ma[-1]` 分别代表了最新的快线和慢线的值，当 `fast_ma` 大于 `slow_ma` 且当前没有仓位（`self.position.size` == 0) 执行买入，否则执行平仓。

**运行回测**

我们要创建 `Backtest` 实例运行回测，将 Google 数据，策略类、初始金额和交易费率传递 `Backtest` 完成实例化。

示例代码：

```python
bt = Backtest(GOOG, MovingAverageCrossStrategy, cash=10000, commission=0.002)
stats = bt.run()
```

运行 `bt.run()`，得到它的返回值 `stats`，这就是回测结果。

```python
print(stats)
```

输出：

```bash
Start                     2004-08-19 00:00:00 # 开始时间
End                       2013-03-01 00:00:00
Duration                   3116 days 00:00:00
Exposure Time [%]                   61.545624
Equity Final [$]                   99485.0574
Equity Peak [$]                   100607.2574
Return [%]                         894.850574
Buy & Hold Return [%]              703.458242
Return (Ann.) [%]                   30.934891
Volatility (Ann.) [%]               32.215003
Sharpe Ratio                         0.960263
Sortino Ratio                        2.125336
Calmar Ratio                         1.497421
Max. Drawdown [%]                  -20.658779
Avg. Drawdown [%]                   -3.678412
Max. Drawdown Duration      584 days 00:00:00
Avg. Drawdown Duration       38 days 00:00:00
# Trades                                   48
Win Rate [%]                        64.583333
Best Trade [%]                       57.11931
Worst Trade [%]                    -12.446769
Avg. Trade [%]                       4.923007
Max. Trade Duration         121 days 00:00:00
Avg. Trade Duration          39 days 00:00:00
Profit Factor                        4.729158
Expectancy [%]                        5.66852
SQN                                  2.643361
_strategy                 MovingAverageCro...
_equity_curve                             ...
_trades                       Size  EntryB...
dtype: object
```


```python
bt.plot()
```



### 结果解读

运行上述代码后，你将看到：

- **策略统计数据**：包括初始资金、最终净值、最大回撤等。
- **交互式回测图表**：图表显示了买卖点、指标和交易统计信息。

在短短 30 行代码内，我们实现了一个简单的 RSI 交叉策略。这是 Backtesting.py 简洁高效的一个典型例子。

## 参数优化

在策略开发中，参数优化是提升回测表现的重要步骤。Backtesting.py 内置了参数优化功能，能够帮助开发者高效找到最佳策略参数组合。

### 使用优化器

以下是一个优化 RSI 策略的示例代码：
```python
class RSIStrategy(Strategy):
    upper_bound = 70
    lower_bound = 30
    rsi_window = 14

    def init(self):
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)

    def next(self):
        if self.rsi[-1] > self.upper_bound:
            self.position.close()
        elif self.rsi[-1] < self.lower_bound:
            self.buy()

# 运行优化
bt = Backtest(GOOG, RSIStrategy, cash=10000, commission=0.002)

stats = bt.optimize(
    upper_bound=range(50, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=range(10, 30, 2),
    maximize='Sharpe Ratio',
    constraint=lambda param: param.lower_bound < param.upper_bound
)
print(stats)
bt.plot()
```

### 优化器参数说明

- **`upper_bound` 和 `lower_bound`**：分别表示 RSI 的超买和超卖阈值。
- **`rsi_window`**：RSI 的时间窗口大小。
- **`maximize`**：优化的目标，可以选择 `Sharpe Ratio`、`Equity Final [$]` 等指标。
- **`constraint`**：添加约束条件，确保参数组合的合理性。

### 并行计算和性能优化

Backtesting.py 支持多线程并行计算，可以充分利用计算机的多核性能：

```python
import os
os.environ['BACKTESTING_NUM_WORKERS'] = '4'  # 使用 4 个线程
```

在运行优化时，Backtesting.py 会并行计算每种参数组合，显著提高效率。

