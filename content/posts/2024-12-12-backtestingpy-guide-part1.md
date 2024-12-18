---
title: "Backtesting.py 由浅入深 Part1: 快速上手使用"
date: 2024-12-18T12:15:18+08:00
draft: false
comment: true
description: "本文将介绍 **Backtesting.py**，一个轻量级的 Python 的交易回测框架。"
---

在算法交易中，验证一个策略是否有效至关重要，但该如何验证呢？

我们可以通过模拟盘或直接实盘测试策略，但能看到的问题有限。最快捷有效的方式是基于历史数据回测（Backtesting），模拟策略在过去市场中的表现，在评估收益和风险后，再进行模拟和实盘。

那该如何回测呢？

回测的方式有很多，既可以手工测试、也可以借助 excel 或是 python 的向量计算库，如 numpy 和 pandas 进行。不过这些方式都比较原始，我们可使用市面一些现成的回测框架，不仅能尽可能避免回测中常见的问题，还能把重点放在策略的逻辑上。

本文将先介绍 **Backtesting.py**，一个 Python 实现轻量级、事件驱动的回测框架。之所以先介绍它，主要是它非常简单。

## 什么是 Backtesting.py

**Backtesting.py** 是一个轻量级的 Python 回测框架，专注于策略回测的核心功能，适合快速实现和测试交易策略。与其他回测框架，如 VectorBT 或 Backtrader 等回测框架相比，**Backtesting.py** 简化了使用流程，没有过多复杂功能，自然地，它更加简单易用与高效。

**Backtesting.py** 内置了策略参数优化工具，可与 Pandas 数据框和 NumPy 数组兼容，易于集成。当然简单是有代价的，它的缺点是不支持多资产交易和分数股交易，这也限制了它的普适性。

让我们快速上手 **Backtesting.py** 的使用吧。

## 安装

确保你已经有了 Python 环境，使用如下命令安装 backtesting.py 和 TA-Lib 两个依赖包。

```bash
pip install backtesting TA-Lib
```

如果 TA-Lib 安装遇到困难，请查看 [TA-Lib 安装指南](https://ta-lib.github.io/ta-lib-python/install.html)，或者也可以选择其他技术指标库，如 pandas-ta。

## 编写第一个策略

开始通过 backtesting.py 实现第一个策略，就以均线交叉策略为例。策略的规则也很简单，就是 SMA 金叉开仓买入、死叉平仓。暂时只考虑做多的情况。

### 导入必要的模块

首先，创建一个名为 `strategy.py` 的 Python 文件，开始编写我们的策略。

导入要用到的模块：

```python
from backtesting import Backtest, Strategy
from backtesting.test import GOOG  # 示例数据
import talib
```

简单起见，我们就用 **Backtesting.py** 提供的样例数据 GOOG，即 Google 某段时间点的价格数据来演示。


### 定义策略类

我们定义一个名为 `MovingAverageCrossStrategy` 策略类，继承 `Strategy`：

```python
class MovingAverageCrossStrategy(Strategy):
  # 参数
  fast_ma_window = 10
  slow_ma_window = 20

  def init(self):
    # 初始化阶段：计算指标
    self.fast_ma = self.I(talib.SMA, self.data.Close, timeperiod=self.fast_ma_window)
    self.slow_ma = self.I(talib.SMA, self.data.Close, timeperiod=self.slow_ma_window)

  def next(self):
    # 交易阶段：逻辑判断和执行交易
    if self.fast_ma[-1] > self.slow_ma[-1] and self.position.size == 0:
      self.buy()
    elif self.fast_ma[-1] < self.slow_ma[-1] and self.position.size > 0:
      self.position.close()
```

在 Backtestingpy 中实现的策略要继承 `backtesting` 下 `Strategy` 类，实现它的两个方法：`init` 和 `next`。`Strategy` 是框架提供的基类，它规定了策略结构和生命周期，init 方法用于初始化，如指标的向量计算，`next` 中用于实现提供数据的每个 bar 的执行逻辑。

策略类上的两个变量 `fast_ma_window` 和 `slow_ma_window` 是我们定义的策略参数，代表了均线的快线和慢线的周期，默认设置为 10 和 20。它们是可配置的，在调用策略时，是可以修改这两个参数的值的。

在 `init` 初始化阶段，我们可以通过 talib 计算 SMA 均线的快慢线。数据可以通过 `self.data` 访问，指标计算用到了收盘价，即 `self.data.Close`。

在 `next` 交易逻辑阶段，只需要拿到当前的指标数据，判断是否要入场就行了，`self.fast_ma[-1]` 和 `self.slow_ma[-1]` 分别代表了最新的快线和慢线的值，当 `fast_ma` 大于 `slow_ma` 且当前没有仓位（`self.position.size` == 0) 执行买入，否则执行平仓。

### 交叉 crossover

如果你希望策略的判断严格是金叉死叉的那一刻，不仅仅要比较 -1，即当前的指标，还有比较 -2，即上个周期的大小。

金叉，即快线 fast_ma 上穿慢线 slow_ma：

```python
fast_ma[-1] > slow_ma[-1] and fast_ma[-2] < slow_ma[-2]
```

死叉，即快线 fast_ma 下穿慢线 slow_ma，也就是慢线 slow_ma 上穿快线 fast_ma：

```python
fast_ma[-1] < slow_ma[-1] and fast_ma[-2] > slow_ma[-2]
```

好在 **Backtesting.py** 提供了一个函数，专门用于判断这种两条线交叉的场景，即 `backtesting.lib` 下 `crossover(s1, s2)` 函数，如果 `s1` 上穿 `s2`，则返回 `True`，否则返回 `False`。


```python
from backtesting.lib import crossover

# 金叉
crossover(fast_ma, slow_ma)
# 死叉
crossover(slow_ma, fast_ma)
```

现在就简洁多了。

### 运行回测

我们要创建 `Backtest` 实例运行回测，将 Google 数据，策略类、初始金额和交易费率传递 `Backtest` 完成实例化。

示例代码：

```python
bt = Backtest(GOOG, MovingAverageCrossStrategy, cash=10000, commission=0.002, slippge)
stats = bt.run()
```

运行 `bt.run()`，得到它的返回值 `stats`，这就是我们要的回测结果。

```python
print(stats)
```

输出评测结果，如下：

```bash
Start                  2004-08-19 00:00:00 # 开始时间
End                    2013-03-01 00:00:00 # 结束时间
Duration                3116 days 00:00:00 # 总持续时间
Exposure Time [%]                61.545624 # 资金暴露时间比例（%）
Equity Final [$]                99485.0574 # 最终净值（美元）
Equity Peak [$]                100607.2574 # 最高净值（美元）
Return [%]                      894.850574 # 总收益率（%）
Buy & Hold Return [%]           703.458242 # 买入持有策略收益率（%）
Return (Ann.) [%]                30.934891 # 年化收益率（%）
Volatility (Ann.) [%]            32.215003 # 年化波动率（%）
Sharpe Ratio                      0.960263 # 夏普比率（风险调整收益）
Sortino Ratio                     2.125336 # 索提诺比率（下行风险调整收益）
Calmar Ratio                      1.497421 # 卡尔玛比率（收益与最大回撤之比）
Max. Drawdown [%]               -20.658779 # 最大回撤（%）
Avg. Drawdown [%]                -3.678412 # 平均回撤幅度（%）
Max. Drawdown Duration   584 days 00:00:00 # 最大回撤持续时间
Avg. Drawdown Duration    38 days 00:00:00 # 平均回撤持续时间
# Trades                                48 # 总交易次数
Win Rate [%]                     64.583333 # 胜率（%）
Best Trade [%]                    57.11931 # 最佳单笔交易收益率（%）
Worst Trade [%]                 -12.446769 # 最差单笔交易收益率（%）
Avg. Trade [%]                    4.923007 # 平均单笔交易收益率（%）
Max. Trade Duration      121 days 00:00:00 # 最长持仓时间
Avg. Trade Duration       39 days 00:00:00 # 平均持仓时间
Profit Factor                     4.729158 # 盈亏比（获利交易总额与亏损交易总额之比）
Expectancy [%]                     5.66852 # 期望收益率（平均每笔交易的收益率）
SQN                               2.643361 # 系统质量数（策略表现综合评分）
_strategy              MovingAverageCro... # 策略名称（移动均线交叉）
_equity_curve                          ... # 净值曲线
_trades                    Size  EntryB... # 交易详情
dtype: object
```

如上是策略统计数据，有包括初始资金、最终净值、最大回撤等，还有其他评价策略表现的指标，如最大回撤、夏普比率、胜率等。

如果想查看回测的收益曲线，`Backtest` 提供方法 `plot` 查看可视化结果：

```python
bt.plot()
```

运行上述代码后，你将看到：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-12/2024-12-12-backtestingpy-guide-part1-01.png)

图表中显示了策略的收益曲线、买卖点、指标曲线和我们重点的几个交易统计数据，如最终净值、最高净值、最大回撤等。

除了使用默认参数外，我们可以在调用 `bt.run()` 修改默认参数：

```python
bt.run(fast_ma_window=15, slow_ma_window=30)
```

在这不到 30 行代码里，我们就实现了一个简单的均线交叉策略，可见 **Backtesting.py** 的简洁高效。

## 自定义数据

**Backtesting.py** 是允许使用自定义数据的。

### 要求格式 

数据要求是 `pandas.DataFrame`，且满足按时间升序排列，时间为 DataFrame 的索引，同时数据列至少包含 Open、High、Low、Close、Volume 五个字段即可。

查看样例数据格式；

```python
print(GOOG)
```

输出：

```bash
              Open    High     Low   Close    Volume
2004-08-19  100.00  104.06   95.96  100.34  22351900
2004-08-20  101.01  109.08  100.50  108.31  11428600
2004-08-23  110.75  113.48  109.05  109.40   9137200
2004-08-24  111.24  111.60  103.57  104.87   7631300
2004-08-25  104.96  108.00  103.88  106.00   4598900
...            ...     ...     ...     ...       ...
2013-02-25  802.30  808.41  790.49  790.77   2303900
2013-02-26  795.00  795.95  784.40  790.13   2202500
2013-02-27  794.80  804.75  791.11  799.78   2026100
2013-02-28  801.10  806.99  801.03  801.20   2265800
2013-03-01  797.80  807.14  796.15  806.19   2175400
```

### 示例：适配 tushare

演示下将 tushare 数据转为 **Backtesting.py** 支持的格式吧。

示例代码：

```python
import tushare as ts
import pandas as pd

pro = ts.pro_api()
df = pro.daily(ts_code="000001.SZ", start_date="2024-01-01")
df["trade_date"] = pd.to_datetime(df["trade_date"])
df.set_index("trade_date", inplace=True)
df.index.name = "Datetime"
df.sort_index(inplace=True)

df.rename(
    columns={
        "open": "Open",
        "high": "High",
        "low": "Low",
        "close": "Close",
        "vol": "Volume",
    },
    inplace=True,
)
df = df[["Open", "High", "Low", "Close", "Volume"]]
print(df.head())
```

输出：

```bash
            Open  High   Low  Close      Volume
Datetime
2024-01-02  9.39  9.42  9.21   9.21  1158366.45
2024-01-03  9.19  9.22  9.15   9.20   733610.31
2024-01-04  9.19  9.19  9.08   9.11   864193.99
2024-01-05  9.10  9.44  9.07   9.27  1991622.16
2024-01-08  9.23  9.30  9.11   9.15  1121156.19
```

## 参数优化

在策略开发时，有一个过程非常重要，即参数优化，它是提升回测表现的重要步骤。**Backtesting.py** 内置了参数优化功能，能助我们快速找到最佳策略参数组合。

### 优化示例

backtestingpy 默认的优化方式是将是将所有的参数组合测试一遍，以下是优化 MovingAverageCross 策略的示例代码：

```python
bt = Backtest(GOOG, MovingAverage, cash=10000, commission=0.002)
stats = bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
)
print(stats)
bt.plot()
```

如上通过 `range` 指定了 `fast_ma_window` 和 `slow_ma_window` 的参数范围，运行的结果就是表现最好的参数的结果。

我们可以通过打印策略实例拿到表现最好的参数。

```python
print(stats._strategy)
print("fast_ma_window", stats._strategy.fast_ma_window)
print("slow_ma_window", stats._strategy.slow_ma_window)
```

输出：

```bash
MovingAverageCrossStrategy(fast_ma_window=5,slow_ma_window=20)
fast_ma_window 5
slow_ma_window 20
```

### 参数约束

上面的优化代码有个明显的不合理，可能出现 `fast_ma_window` 小于 `slow_ma_window` 的情况，如 `fast_ma_window=20`，但 `slow_ma_window=10` 的组合，这不仅会增加优化耗时，也不符合策略的逻辑，。`backtestingpy` 的优化器提供了参数约束能力，我们可以限制 `fast_ma_window` 必须小于 `slow_ma_window`。

示例代码：

```python
stats = bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
    constraint=lambda p: p.fast_ma_window < p.slow_ma_window,
)
```

这将能极大提升回测优化速度。

### 评价标准

优化器默认只返回了一个表现最好的参数，但这个最好是如何评价的呢？

backtestingpy 优化器的默认评价标准是 SQN，一句话表述它的含义就是，SQN 通过平均收益（高更好）、收益波动性（低更好）和交易次数（多更好）衡量策略稳健性，值越高越优质。

假设，我想修改这个评价可以吗？当然可以。`optimize` 提供了一个名为 `maximize`参数来修改评价标准。我可以将最大回撤或夏普比率作为评价标准。

将评价标准改为最大回撤：

```python
bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
    constraint=lambda p: p.fast_ma_window < p.slow_ma_window,
    maximize='Max. Drawdown [%]',
)
```

将评价标准改为夏普比率：

```python
bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
    constraint=lambda p: p.fast_ma_window < p.slow_ma_window,
    maximize="Sharpe Ratio",
)
```

这个评价标准默认是选设置的评价指标的最大值，如果想越小越好，可通过传递函数实现，如果选择年化波动率最小的参数组合。

示例代码：

```python

bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
    constraint=lambda p: p.fast_ma_window < p.slow_ma_window,
    maximize=lambda r: -r["Volatility (Ann.) [%]"],
)
```

通过 `maximize=lambda r: -r["Volatility (Ann.) [%]"]` 选择年化波动波动率最小的组合。

不过，从我平时的使用体验来看，还是默认的 SQN 的优化结果比较合理，它的评价维度更加全面。

## 回测图表保存

**Backtesting.py** 提供了保存回测图表的功能，便于后续分析。

### 保存 HTML 文件

我们上面运行回测时，**Backtesting.py** 会在当前目录下自动生成一个 HTML 文件，即上面展示的回测图表。

默认的文件名称是 "策略类名.html"，参数优化的话，文件名会带上参数。

示例：

```bash
MovingAverageCrossStrategy(fast_ma_window=5,slow_ma_window=20).html
MovingAverageCrossStrategy.html
```

这个名称是可以修改的：

```python
bt.plot(filename='results/plot.html')
```

上述代码将回测结果保存在 `results` 目录下。如果目录不存在，需要提前创建：

```bash
mkdir -p results
```

### 动态命名文件

我们也可以动态设置文件名以区分不同的参数组合。例如：

```python
fast =  stats._strategy.fast_ma_window
slow = stats._strategy.slow_ma_window
bt.plot(
  filename=f'results/fast{fast}_slow{slow}.html',
)
```

这将保存文件名如 `fast10_slow20.html` 的 HTML 文件，方便管理不同的回测结果。

## 总结

本文介绍了 **Backtesting.py** 回测框架的快速上手使用，从策略创建、回测与参数优化。**Backtesting.py** 的参数优化部分还有不少的内容，后续文章再继续介绍。
