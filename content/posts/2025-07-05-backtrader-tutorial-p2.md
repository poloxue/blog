---
title: "Backtrader 教程二：安装与快速开始"
date: 2025-07-05T10:02:36+08:00
draft: false
---

本文将一步步完成 Backtrader 的安装，并通过两个简单策略——买入持有与均线交叉策略，带你快速熟悉 Backtrader 的结构与用法。

## 快速安装

Backtrader 的安装过程非常简单，即使你是刚接触 Python，也能快速搞定。

打开你的命令终端（比如 Windows 的 CMD 或 Mac 的终端），敲入这行代码：

```bash
pip install backtrader
```

输入 "Enter" 确认就能很快安装就完成。

## 简单例子

先通过一个最小化的可运行示例，模拟用历史数据加载苹果股票（AAPL），并通过 Backtrader 运行并画图。

这个例子不带任何交易策略，仅用于感受框架整体流程。

```python
import backtrader as bt
from datetime import datetime
import yfinance as yf
import pandas as pd

# 使用 yfinance 下载苹果股票数据并转换为 Pandas DataFrame
raw_data = yf.download('AAPL', start='2020-01-01', end='2021-01-01', multi_level_index=False)
raw_data.reset_index(inplace=True)

# 转换为 Backtrader 可识别的 Pandas 数据源
apple_data = bt.feeds.PandasData(dataname=raw_data)

cerebro = bt.Cerebro()
cerebro.broker.setcash(1e8)
cerebro.adddata(apple_data)

print(f"Initial portfolio value: {cerebro.broker.getcash()}")
cerebro.run()
print(f"Final portfolio value: {cerebro.broker.getcash()}")

cerebro.plot()
```

这将生成一个图表，显示价格走势与策略执行情况，能让你第一时间看到回测结果。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-07/2025-07-05-backtrader-tutorial-p2-01.png)

图表上的资产净值从开始就没有变化，因为这个例子中没有添加任何的策略。

简单介绍下这段代码吧。

* 第一步，创建回测引擎并设置初始资金：

```python
cerebro = bt.Cerebro()
cerebro.broker.setcash(1e8)
```

cerebro 是整个大脑，负责管理策略逻辑、数据源、资金、回测过程等。

* 第二步，加载数据，以 apple 股票数据为例：

```python
apple_data = bt.feeds.PandasData(dataname=raw_data)
cerebro.adddata(apple_data)
```

这里使用的是 yfinance 下载的 DataFrame 数据，并通过 `PandasData` 转为 Backtrader 所能识别的数据格式。

* 第三步，执行回测：

```python
cerebro.run()
```

Backtrader 会从头到尾执行策略逻辑，即使此时你没有写具体的交易逻辑，它也会完整遍历数据。

* 最后一步画图展示：

```python
cerebro.plot()
```

这个例子没有写一行策略代码就能跑起来，很适合初学者快速上手。

## 买入持有策略

开始用 backtrader 回测一个最简单的策略：买入持有。

买入并持有是一个朴素的投资理念，很多人会说：“如果一直持有不卖，现在早赚翻了！”。尝试用 backtrader 的代码实现这个简单的投资逻辑。

```python
class BuyHold(bt.Strategy):
    def next(self):
        if not self.position:
            self.buy()

cerebro = bt.Cerebro()
cerebro.addstrategy(BuyHold)

cerebro.adddata(apple_data)
cerebro.broker.setcash(100000)

print(f"Initial portfolio value: {cerebro.broker.getcash()}")
cerebro.run()
print(f"Final portfolio value: {cerebro.broker.getcash()}")
cerebro.plot()
```

输出：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-07/2025-07-05-backtrader-tutorial-p2-02.png)

执行策略，就可以清楚看到，策略在第一个交易日买入股票后就再也没有新的动作，账户资金和股价变化基本保持一致。

理解三个点：

* 第一，策略是通过 `cerebro.addstrategy()` 这句代码加载进回测引擎的，这就像是把你的交易逻辑交给“大脑”去执行；
* 第二，策略本身的逻辑是通过继承 `bt.Strategy` 并实现 `next()` 方法来定义的。

  Backtrader 每处理一根 K 线数据，都会自动调用策略的 `next()` 方法。这个方法就像策略的“心跳”，交易逻辑都在这里。
* 第三，策略实现部分，通过 `self.position` 判断当前是否持仓，是最基础也最重要的条件判断语句之一。

  当你写下 `if not self.position: self.buy()` 时，已经完成了一个完整的量化判断过程：检查当前是否有仓位，然后在满足条件时买入。

这个代码整体既直观，又具备强大的扩展能力。接下来，再尝试实现一个更加复杂的策略吧！

## 均线交叉策略

在交易策略里，“金叉买入，死叉卖出”绝对是老生常谈。但真实情况如何呢？

通过 Backtrader 亲自测试下吧。

如下是均线交叉策略的实现：

```python
class SMACross(bt.Strategy):
    def __init__(self):
        # 初始化时定义两个均线指标：快速和慢速
        self.fast_ma = bt.indicators.SMA(period=10)
        self.slow_ma = bt.indicators.SMA(period=30)

    def next(self):
        # 每根K线都会调用此方法，执行一次策略判断逻辑
        if not self.position:
            # 没持仓时，若发生金叉（快线由下向上穿过慢线），则买入
            if self.fast_ma[0] > self.slow_ma[0] and self.fast_ma[-1] <= self.slow_ma[-1]:
                self.buy()
        else:
            # 已持仓时，若发生死叉（快线由上向下穿过慢线），则卖出
            if self.fast_ma[0] < self.slow_ma[0] and self.fast_ma[-1] >= self.slow_ma[-1]:
                self.sell()
```

输出：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-07/2025-07-05-backtrader-tutorial-p2-03.png)

直观感受到“金叉买入，死叉卖出”的效果到底如何，收益和亏损都会清晰地展现了出来。

这个策略用 backtrader 内置技术指标 `bt.indicators.SMA` 计算移动平均线。

在 next() 方法中，通过判断当前与前一根K线的均线关系，识别出“金叉”与“死叉”信号。

如果想简化 `if` 金叉死叉的判断条件，还可以通过 `bt.ind` 的 `CrossUp` 和 `CrossDown` 计算是否发生了交叉。

```python
self.crossup = bt.ind.CrossUp(self.fast_ma, self.slow_ma)
self.crossdown = bt.ind.CrossDown(self.fast_ma, self.slow_ma)
```

买入条件就是：

```python
self.crossup[0]
```

卖出条件就是：

```python
self.crossdown[0]
```

Backtrader 指标系统简化了策略实现过程，提高了可读性稳定性。

现在只要将 `cerebro.addstrategy` 中 `BuyHold` 策略修改为 `SMACross` 即可用新策略回测了。


## 六、简单介绍分析指标

在使用 Backtrader 时，你还可以轻松添加各种分析指标，比如年化收益率、夏普比率、最大回撤等。这些指标能帮你快速、清楚地评估策略表现，从而更科学地判断策略好坏。

```python
cerebro.addanalyzer(bt.analyzers.SharpeRatio, _name='sharpe')
results = cerebro.run()
sharpe_ratio = results[0].analyzers.sharpe.get_analysis()
print("夏普比率：", sharpe_ratio)
```

Backtrader 内置了大量分析工具可以直接调用。例如：

* `DrawDown`：查看回撤表现，衡量风险暴露
* `Returns`：计算每日或周期收益率
* `TradeAnalyzer`：输出每一笔交易的盈亏、胜率等细节
* `SQN`：系统质量指数（System Quality Number），衡量策略整体表现
* `TimeReturn`：用于时间序列的收益计算

这些分析器都可以像 Sharpe 一样通过 `addanalyzer()` 方法添加，并通过 `get_analysis()` 获取结果。

有了这些分析指标，你的交易评估会更加全面和精准。使用得当，它们能帮助你对策略进行多维度评价，辅助决策与优化。

## 总结

本文从 Backtrader 的安装开始，带你跑通了最基础的回测流程。

从加载数据、创建回测引擎，到实现两个基础策略（买入持有与均线交叉），基本过了下 Backtrader 基本结构和用法。

此外，还初步接触了如何使用分析器评估策略表现。

希望这些内容能助你对 Backtrader 有了初步的认识，建立起对量化回测的整体感知。
