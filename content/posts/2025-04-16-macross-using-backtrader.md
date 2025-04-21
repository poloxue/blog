---
title: "掌握 backtrader 策略回测：均线交叉交易系统"
date: 2025-04-15T17:59:11+08:00
draft: false
comment: true
---

我花了一个星期通读了 backtrader 的官方文档，順便还通过 AI 整理出了它的中文文档，我不想写 backtrader 的基础入门教程，还不如直接用 backtrader 逐步展开回测这一个策略，或许更加容易掌握它的使用。

相较于其他回测框架，backtrader 的功能强大，除了支持单品种，还可以实现组合策略，扩展能力非常强大，如果说缺点，一是这个项目基本不维护了，不过不影响它的使用，还有就是因为它很强大，扩展性强，于比其他的回测更不易于掌握。

本文介绍如何使用 backtrader 回测双均线交易策略，将包含基本的进出场、风险控制，参数优化等，还有如何加载分钟数据尽可能模拟真实场景。我暂时先用双均线策略上手 backtrader，后续还会使用 backtrader 回测更多的策略。

## 策略定义

**选择品种**

- 暂定 BTC 作为回测品种，虽然这个策略在它上面的表现不算很好。

**技术指标**

- EMA，即指数移动均线，均线指标种类繁多，如 SMA、WMA 等；
- ATR，真实波动幅度，用于控制止损；

**策略参数**

- short_period：短均线周期；
- long_period：长均线的周期
- atr_period： 真实波动振幅的计算周期，一般固定为 14 即可。

**入场条件**

- 做多：短均线上穿长均线。
- 做空：短均线下穿长均线。

**出场条件**

- 平多：短均线下穿长均线。
- 平空：短均线上穿长均线。

**风险控制**

为了防止出现过于大的止损金额，设置固定的止损位置并通过止损位置推算最大亏损金额。

- 止损位置：当前价格 - n 倍 ATR；
- 最大亏损：每笔交易的最大亏损金额不超过账户的 2%，按次目标推算实际下单金额。

这是个非常简单的策略，只是相对于最简单版本的均线交叉策略，加上了风险控制。

## 依赖库

安装依赖的 Python 库。

```bash
pip install backtrader yfinance
```

## 主流程

通过 backtrader 回测策略，在主函数要提前做一些设置，如设置费率、滑点、账户金额。此外还有加载数据源和策略。


```python
import backtrader as bt
import yfinance as yf


# 策略逻辑，暂空
class MACrossStrategy(bt.Strategy):
  pass


if __name__ == "__main__":
  cerebro = bt.Cerebro()

  # 数字货币交易 taker 的手续费一般是这个数字。
  cerebro.broker.setcommission(0.0005) 
  cerebro.broker.set_slippage_perc(0.0001)

  # 初始资金为 1百万个货币单位
  cerebro.broker.setcash(1e6)

  data = yf.download(tickers='BTC-USD',
                     start='2020-01-01',
                     end='2025-04-16',
                     interval='1d',
                     multi_level_index=False)
  cerebro.adddata(bt.feeds.PandasData(dataname=data))

  cerebro.addstrategy(MACrossStrategy)

  print("初始净值：", cerebro.broker.getvalue())
  cerebro.run()
  print("最终净值：", cerebro.broker.getvalue())

  cerebro.plot()
```

backtrader 中的 `Cerebro` 类型是核心大脑，其他的组件都是围绕着它进行配置。

这段代码的核心内容包括：

- 设置费率为万五，滑点为万1，初始资金为 1e6，即 1百万货币单位。
- 开始回测前，添加标的数据，通过 `yfinance` 下载数据，如加载 `BTC-USD` 的历史行情。
- 接着加载要回测的策略 `MACrossStrategy`，继承自 `bt.Strategy`。
- 最后运行回测并输出策略前后的账户净值数据并绘制图。

策略还没有实现任何的交易逻辑，如果运行回测，你会看到前后的账户资产没有变化。

## 策略实现

实现一个策略有两个步骤，分别是计算指标和策略逻辑两个部分，分别对应于 `MACrossStrategy` 的 `__init__` 和 `next` 函数。

### 计算指标

计算这个策略用到的三个指标，分别是 `short_ma`、`long_ma` 和 `atr`。

```python
class MACrossStrategy(bt.Strategy):
  params = (
      ("short_period", 10),
      ("long_period", 20),
      ("atr_period", 14),
  )
  def __init__(self):
    self.short_ma = bt.ind.EMA(period=self.p.short_period)
    self.long_ma = bt.ind.EMA(period=self.p.long_period)
    self.atr = bt.ind.ATR(period=self.p.atr_period)

    # 短均线上穿长均线
    self.crossover = bt.ind.CrossUp(self.short_ma, self.long_ma)
    self.crossunder = bt.ind.CrossDown(self.short_ma, self.long_ma)

```

这里将计算指标要用到的周期参数化，以便于后续优化。而且为了便于后续交易逻辑判断，通过 backtrader 的 `crossover` 和 `crossunder` 辅助函数判断均线的金叉死叉。

backtrader 支持的指标繁多，可通过几行代码拿到支持的常见指标：

```python
indicators = [attr for attr in dir(bt.ind) if not attr.startswith('_')]
print("Backtrader支持的内置指标:")
for idx, indicator in enumerate(sorted(indicators), 1):
    print(f"{idx}. {indicator}")
```

输出：

```bash
Backtrader支持的内置指标:
1. ADX
2. ADXR
3. AO
4. APO
5. ATR
...
```

如果熟悉 talib，也可以通过使用它。backtrader 内置支持了 talib。

```bash
short_ma = bt.talib.EMA(timeperiod=self.p.short_period)
```

### 开仓平仓

交易逻辑部分可分为开仓平仓和风险控制两部分。简单的开仓平仓实现起来较为简单，先介绍这部分的实现。而风险控制要依赖条件单实现，backtrader 是支持条件单的。

开仓平仓逻辑的代码如下所示：

```python
class MACrossStrategy(bt.Strategy):

  def __init__(self):
    pass

  def next(self):
    long_entry = (
      self.crossup[0] == 1
      and self.position.size <= 0
    )
    short_entry = (
      self.crossdown[0] == 1
      and self.position.size >= 0
    )

    long_exit = (
      self.crossdown[0] == 1
      and self.position.size > 0
    )
    short_exit = (
      self.crossup[0] == 1
      and self.position.size < 0
    )

    if long_entry:
        self.order_target_percent(target=0.99)
    elif short_entry:
        self.order_target_percent(target=-0.99)

    if long_exit:
        self.close()
    elif short_exit:
        self.close()
```

按设定规则开平仓，如果短线上穿长线开多，短线下穿长线开空，平仓的条件反之。

下单方式没有使用 `buy` 和 `sell` 方法，直接用了 backtrader 提供的目标订单方法，省掉了下单数量的计算。如果想拿到直接下单大小，`order_target_percent` 返回了订单 `Order`，有订单大小 `size` 属性，这个后面设置止损条件是要用到的。

backtrader 有个非常重要的 Line 的概念，相对是比较复杂的，这里不展开说了。记住的是，如果要访问当前数据，下标都是 0 开始的，往前看就是 -1、-2，以此类推。所以，`self.crossup[0]` 就是检查当前短均线是否上穿长均线。

暂时这里还没有加入风险控制，直接 0.99 的仓位，基本就是全仓了。 

现在可以运行下这个回测。

```bash
python main.py
```

输出：

```bash
初始净值： 1000000.0
最终净值： 3280739.0840676297
```

五年多的时间赚了 3 倍收益，和直接持有 BTC 的这五年收益相比，简直是浪费时间。

> 完整代码请访问：[双均线策略基础版本](https://github.com/poloxue/backtest_strategies/blob/main/movingaverage/backtest/basic.py)。

## 绘图

回测结束除了可以拿到最终净值，运行 `cerebro.plot()` 还输出一张图，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-01.png)

这个图中有几个部分，分别是净值曲线、交易盈亏点、价格蜡烛图（包括买卖点标注）和技术指标。

这个图的收益部分显示效果不是很清晰，如果想只看收益曲线，可通过配置 `analyzers` 拿到收益序列。

```python

cerebro.addanalyzer(bt.analyzers.TimeReturn, _name="timereturn")

strat = cerebro.run()

returns = strat[0].analyzers.getbyname('timereturn').get_analysis()
returns_series = pd.Series(returns)
net_value = (1 + returns_series).cumprod()
ax = net_value.plot(title="Returns", figsize=(12, 5))

end_value = net_value.iloc[-1]
end_index = net_value.index[-1]

ax.text(
    end_index,
    end_value,
    f"End: {end_value:.2f}",
    ha="right",
    va="top",
    bbox=dict(facecolor="white", alpha=0.8),
)

plt.show()
```

净值曲线如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-02-v1.png)

其他数据，如行情、指标、交易记录等，然也可以拿出来单独绘图。

> 完整代码请访问：[自定义收益曲线](https://github.com/poloxue/backtest_strategies/blob/main/movingaverage/backtest/plot_timereturns.py)。

## 评价指标

从上面的净值曲线图，这个策略的回撤非常大。如果想看最大回撤的具体大小，要添加分析器。夏普比率是同样的思路。

```python
cerebro.addanalyzer(bt.analyzers.DrawDown, _name="drawdown")
cerebro.addanalyzer(bt.analyzers.DrawDown, _name="shape_ratio")
strat = cerebro.run()

drawdown = strat[0].analyzers.getbyname("drawdown").get_analysis()
sharpe_ratio = strat[0].analyzers.getbyname("drawdown").get_analysis()
print("最大回撤", drawdown["max"]["drawdown"])
print("夏普比率", shape_ratio["shape_raito"])
```

输出：

```bash
最大回撤 82.12757289556565
夏普比率 0.5393603817365935
```

其他常见的评价指标，基本都可通过配置分析器拿到，如 SQN 指标，这个常用来选择参数优化的最优策略。

```python
cerebro.addanalyzer(bt.analyzers.SQN, _name="sqn")
```

## 策略优化

暂时先不加入风险控制，尝试优化下双均线策略。

backtrader 内置网格搜索优化的函数。将前面的 `addstrategy` 换成 `optstrategy`，指定参数搜索的范围。

```python
cerebro.optstrategy(
  MACrossStrategy,
  short_period=range(5, 50, 5),
  long_period=range(20, 200, 10),
)
```

为防止不满足条件的参数出现，如 short_period >= long_period，修改下策略实现：

```python
class MACrossStrategy(bt.Strategy):
  def __init__(self):
    if self.p.short_period >= self.long_period:
      raise bt.StrategySkipError()
```

这样也能提升优化的效率。

```python
opt_returns = cerebro.run(optreturn=False)
```

运行后，将得到一个可遍历列表 `opt_returns`，它的元素是每个参数组合的数据。

```python
for opt_return in opt_returns:
  if len(ret):
    print("参数：", ret[0].params.short_period, ret[0].params.long_period)
    print("净值：", ret[0].broker.getvalue())
    print("夏普：", ret[0].analyzers.sharpe.get_analysis())
```

现在通过 for 输出各个参数组合的回测结果。

上面之所以要设置 `optreturn=False` 是为了拿到策略的完整数据，否则如 `.broker.getvalue()` 是无法访问的。

Backtrader 的优化功能和 backtesting.py 不同，它的内置优化功能（cerebro.optstrategy() 和 cerebro.run()）不会自动选择最优策略。它只会执行所有参数组合的回测，并返回每个组合的结果。我还需要分析这些结果并选择最佳参数。

如选择夏普比率最优的参数组合：

```python
opt_returns = cerebro.run(optreturn=False)
best_strategy = None
for ret in opt_returns:
    if len(ret):
        if best_strategy is None:
            best_strategy = ret[0]
        else:
            best_sharpe_ratio = best_strategy.analyzers.sharpe.get_analysis()[
                "sharperatio"
            ]
            sharpe_ratio = ret[0].analyzers.sharpe.get_analysis()["sharperatio"]
            if best_sharpe_ratio < sharpe_ratio:
                best_strategy = ret[0]

short_period = best_strategy.params.short_period
long_period = best_strategy.params.long_period
sharpe_ratio = best_strategy.analyzers.sharpe.get_analysis()["sharperatio"]
max_drawdown = best_strategy.analyzers.drawdown.get_analysis()["max"]["drawdown"]
final_value = best_strategy.broker.getvalue()

print(f"参数组合：short {short_period}, long {long_period}")
print(f"夏普比率：{sharpe_ratio}")
print(f"最大回撤：{max_drawdown}")
print(f"最终净值：{final_value}")

```

输出：

```bash
参数组合：short 10, long 90
夏普比率：0.9247734042607272
最大回撤：57.21188766636116
最终净值：12699593.957906708
```

你可以设定任意的标准来定义什么是最佳参数组合，backtesting.py 框架默认采用选择 SQN 作为选择评价最优策略标准，backtrader 也可以实现，分析加入 SQN 即可。

```python
cerebro.addanalyzer(bt.analyzers.SQN, _name="sqn")
```

优化就展开这么多吧！

> 完整代码请访问：[参数优化](https://github.com/poloxue/backtest_strategies/blob/main/movingaverage/backtest/optimize_parameters.py)。

## 风险控制

到现在策略就是简单通过均线金叉死叉进行开平仓，完全按照这个标准交易，有可能出现损失过大。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-03.png)

如上图所示，短线上穿长线开仓做多后，开始快速下跌，从开仓到死叉平仓的损失不可能控，有可能导致异常的损失。

我想通过设置止损价格解决这个问题，为防止被假止损，采用 n * ATR 计算止损位置。ATR 指标是个波动指标，如果波动太大可能导致不可控的损失。可基于最大损失百分比去反向推算下单金额，如每次最大损失不超过 2%。

假设 n = 3，在策略中实现这个逻辑，示例代码如下：

```python
target_percent = 0.02 / (3 * self.atr[0] / self.data.close[0])
```

用这个值替代之前每次满仓的 0.99，实现每单的最大损失为总资产的 2%。

多仓的止损价格：

```python
stop_price = self.data.close[0] - 3 * self.atr[0]
```

止损单通过 backtrader 的 stop 类型订单实现。

基于开仓订单创建止损订单：

```python
order = self.order_target_percent(target=target_percent)
self.stop_sell_order = self.sell(
    size=order.size, exectype=bt.Order.Stop, price=stop_price
)
```

开空单同理：

```python
order = self.order_target_percent(target=target_percent)
self.stop_buy_order = self.sell(
    size=order.size, exectype=bt.Order.Stop, price=stop_price
)
```

这里有个问题，如果仓位平掉后，记得要取消这个止损单，可通过 backtrader 的 `notify_trade` 回测检查仓位，如发现仓位非做空仓位，取消 `stop_buy_order`。

现在，默认参数（short_period=10，long_period=20）运行回测，净值曲线如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-04-v1.png)

可以对比下最初的净值曲线：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-02.png)

这里只是增加了一个基本的风险控制，现在的净值曲线比之前稳定了不少。

> 完整代码请访问：[风险控制](https://github.com/poloxue/backtest_strategies/blob/main/movingaverage/backtest/risk_control.py)。

但这还有明显的大涨大跌，如果已经设置了最大损失百分比，应该不至于出现还有这么可怕的波动。

初步估计这个原因可能是行情数据用的日线数据，价格跳动太大导致的滑点严重。而一旦回撤太大，后续想要翻身的难度也会增加。

接下来，尝试用 backtrader 加载分钟数据降低滑点。

## 重放分钟行情

backtrader 提供了数据重放 `replaydata` 加载数据，实现近乎模拟真实的场景。重放 `replaydata` 通过重放高频数据，事实生成某个周期的行情，如日线、4 小时等，即使这个周期还没闭合。

我提前下来了 BTCUSDT 的 1分钟数据，将其通过 `replaydata` 加载到 `cerebro` 中。

```python
data = pd.read_csv(
  "btcusdt_1m",
  parse_dates=['datetime'],
  index_col=['datetime'],
)
cerebro.replaydata(
  data,
  timeframe=bt.TimeFrame.Minutes,
  compression=1440,
)
```

这个地方有一个注意点，如果原始数据是分钟级别的数据，`replaydata` 的 timeframe 只能是 `Minutes`，通过 `compression` 指定合成日线所需的分钟数据量（一天等于 60 * 24 = 1440）。如果是其他周期，如 `bt.TimeFrame.Days`，重放是无法合成日线数据的。

策略逻辑的判断部分也有要修改的地方：

```python
long_entry = self.crossup[-1] and self.position.size <= 0
short_entry = self.crossdown[-1] and self.position.size >= 0

long_exit = self.crossdown[-1] and self.position.size > 0
short_exit = self.crossup[-1] and self.position.size < 0
```

将原来的获取最新的指标修改为上个周期的指标，原因当然就是，当前的 bar 还没有闭合。

这个回测耗时相当长，最终得到的回测净值曲线：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-05.png)

从上图看出，通过分钟回测，得到的净值曲线更平缓，虽然从 2022 年以后这个策略不怎么赚钱了，但不至于那么过分的大涨大跌，产生过大的风险。

这里还有个小问题，如果看上图，偶尔有几个月空仓的情况，那是因为如果是止损单平仓，而非出现金叉死叉触发平仓，可能导致无法重新入场导致丢失掉大趋势。

重新修正下开仓的判断条件。

```python
if self.crossup[-1] == 1:
    self.direction = "long"
elif self.crossdown[-1] == 1:
    self.direction = "short"

long_entry = (
    self.direction == "long"
    and self.data.close[-1] > self.short_ma[-1] > self.long_ma[-1]
    and self.position.size <= 0
)
short_entry = (
    self.direction == "short"
    and self.data.close[-1] < self.short_ma[-1] < self.long_ma[-1]
    and self.position.size >= 0
)
```

在出现金叉死叉时才切换下单方向。这个就不想过多解释了。

净值曲线如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-04/2025-04-16-macross-using-backtrader-06.png)


> 完整代码请访问：[重放模拟回测](https://github.com/poloxue/backtest_strategies/blob/main/movingaverage/backtest/replaydata.py)。

## 总结

本文介绍了如何用 backtrader 回测双均线策略，一步步展开。虽然是一个简单的均线策略，但如果无法正确利用回测框架，可能产生的结果差异较大。

这个策略表现的并不算好，5 年时间在 BTC 上只有不到 4 倍的收益，有兴趣的话，可继续优化，如将日线切换为小时线，或许会有新的发现哦。

> 最后特别说明：本文是我用 backtrader 回测双均线的记录，具体代码在文中都有提供，或许有错误，请仔细甄别。

最后，希望本文对你学习 backtrader 策略回测有所帮助。

