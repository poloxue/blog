---
title: "用 backtrader 回测动量策略"
date: 2025-12-02T02:11:23+08:00
draft: false
comment: true
description: "在金融市场，有一种被称为 动量 的现象。如果一个资产（如股票）的价格正在上涨，那么它短期内更有可能继续上涨；同样，如果一个资产的价格正在下跌，它也更有可能继续下跌。"
---

在金融市场，有一种被称为 "动量" 的现象。其核心理念很简单：如果一个资产（如股票）的价格正在上涨，那么它短期内更有可能继续上涨；同样，如果一个资产的价格正在下跌，它也更有可能继续下跌。这就像惯性一样。

本文将基于这个现象具体介绍下动量策略，并回测这个策略在 数字货币市场上的表现。

## 前言说明

我们先搞清楚为什么会出现动量。常见的说法有两个主要原因：

**信息传播需要时间**：当一条好消息或坏消息出现时，并非所有市场参与者都能立刻知晓并采取行动，价格的调整会随着信息的逐步扩散而持续一段时间。

**机构交易行为**：大型投资机构在买卖大量资产时，会不断执行买入操作，这个交易行为本身就会在市场上形成一个持续的趋势。

基于此，发展出了两种主要的动量策略思路：

**时间序列动量**：关注资产自身的历史表现。例如，买入过去一段时间表现**绝对**好的资产，卖出表现**绝对**差的资产。

**横截面动量**：关注资产之间的**相对**表现。例如，在一个股票组合中，买入过去一段时间表现**最好**的几支，同时卖出表现**最差**的几支，形成一个“做多强势、做空弱势”的组合。

基于这个策略架构，我将用 backtrader 测试动量策略在币圈的表现情况。

## 策略定义

首先选择交易的池子，我将选择市值前 10 的加密货币作为交易标的，不包括 USDT 和其他的包装代币。

交易逻辑采用横截面动量思路，选择每周内表现好的币种做多，同时做空表现差的币种，比例是五五开。这其实是实现了对冲，算是个中性策略，要保证多仓和空仓的价值相等。

还有，即使每周的多空品种没有变化，也会再平衡，实现每个品种的风险暴露相同。

因为没找到按日期查询前十市值数字货币的方法，我就按当前时间固定了 10 个币种。这里其实是引入了未来信息了。

如果查看 [CoinMarketcap 的历史快照页面](https://coinmarketcap.com/historical/)，历史上的前 10 市值的品种变化不算太大。

## 回测代码

策略代码实现起来非常简单，如下是策略的核心交易逻辑部分。

```python
import backtrader as bt
import backtrader.indicators as btind

class MomentumStrategy(bt.Strategy):
    def __init__(self):
        self.pchgs = {}
        for data in self.datas:
            self.pchgs[data] = bt.ind.PercentChange(data, period=1)

        self.count = len(self.datas)
        self.cut_pos = int(self.count / 2)

    def next(self):
        sorted_datas = sorted(
            self.pchgs.keys(),
            key=lambda k: self.pchgs[k][0],
        )

        total_value = self.broker.getvalue()
        target_value = total_value / self.count
        for data in sorted_datas[:self.cut_pos]:
            self.order_target_value(data, -target_value)

        for data in sorted_datas[-self.cut_pos:]:
            self.order_target_value(data, target_value)
```


指标是通过 backtrader 的 PercentChange 计算，period 设置为1。这样在我们加载周线数据到系统就行了。

```python
self.pchgs = {}
for data in self.datas:
    self.pchgs[data] = bt.ind.PercentChange(data, period=1)
```

因为这是一个组合策略，有多个标的，指标是通过字典，键是 backtrader 的 dataFeed 对象，代表的具体的某个品种。

在 next 策略逻辑方法中，按每周变化对各个标的排序。

```python
sorted_datas = sorted(
    self.pchgs.keys(),
    key=lambda k: self.pchgs[k][0],
)
```

接着就是每周定期按当前的持仓价值，在在平衡风险暴露，统一风险暴露。

```python
total_value = self.broker.getvalue()
target_value = total_value / self.count
for data in sorted_datas[:self.cut_pos]:
    self.order_target_value(data, -target_value)

for data in sorted_datas[-self.cut_pos:]:
    self.order_target_value(data, target_value)
```

到这里，就核心的策略部分，每个周期按当前组合总价值对组合进行平衡。

接下来是主函数部分。

```python
symbols = [
    "BTC/USDT",
    "ETH/USDT",
    "XRP/USDT",
    "BNB/USDT",
    "SOL/USDT",
    "TRX/USDT",
    "DOGE/USDT",
    "ADA/USDT",
    "BCH/USDT",
    "LINK/USDT",
]
cerebro = bt.Cerebro(stdstats=False)
# 只绘制 broker 资金和价值变化
cerebro.addobserver(bt.observers.Broker) 

cerebro.broker.setcash(1e8)
cerebro.broker.setcommission(commission=0.0005)
cerebro.broker.set_slippage_perc(0.0001)

cerebro.addanalyzer(bt.analyzers.drawdown.DrawDown, _name="drawdown")

if len(symbols) % 2 != 0:
    raise ValueError(f"标的个数是{len(symbols)}, 要保证能匹配成对")
```

选择了市值排名前十的标的，设置初始资金为1e8，费率为万五，滑点为万一，还加入了回撤 Drawdown 分析器。

如下加载数据和策略并运行回测代码。

```python
for symbol in symbols:
    df = download(
        symbol, start_date="2020-01-01", end_date="2025-11-30", interval="1w"
    )
    data = bt.feeds.PandasData(
        dataname=df,
        name=symbol,
    )
    data.plotinfo.plot = False
    cerebro.adddata(data)

cerebro.addstrategy(MomentumStrategy)

print(f"初始持仓价值：{cerebro.broker.getvalue()}")
strats = cerebro.run()
print(f"最终持仓价值：{cerebro.broker.getvalue()}")
max_drawdown = strats[0].analyzers.getbyname("drawdown").get_analysis()['max']['drawdown']
print(f"最大回撤：{max_drawdown}")

cerebro.plot()
```

运行结果：

```bash
初始持仓价值：100000000.0
最终持仓价值：605325844.8474668
最大回撤：17.627636120733097
```

绘图结果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-12/2025-12-02-moment-strategy-01.png)

看起来还不错，回撤 17% 也不是很大。不过这里用的周线数据，更细粒度的回撤没有被回测出来。

还有这里没有风险控制，可以用日线或小时线回测看看，这里应该还有优化空间。

数据下载函数 download 是我通过 ccxt 封装的函数，这里不暂时出来了。完整代码请查看地址 [momentum.py](https://github.com/poloxue/strategy_backtest/blob/main/momentum.py)。

还有一点注意，加入的多个标的数据要保证日期对齐，否则会导致回测结果错乱。

## 最后

本文是基于动量这个思路的策略回测，如果从周线上看，这个动量效应还是比较明显的，或许这和高杠杆有很大关系吧。
