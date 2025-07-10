---
title: Backtrader 教程三： 从策略生命周期看 backtrader 策略开发
date: 2025-07-06T16:02:14+08:00
draft: true
comment: false
---

在使用 Backtrader 编写策略时，理解 bt.Strategy 策略类的整体结构和生命周期，是核心的第一步。

一个策略不仅仅是几个下单判断，还是一套完整的运行流程，从初始化、数据准备（包括指标和策略运行过程中的状态数据）、执行判断，到响应交易结果与总结，每一步都承担着独立而明确的职责。

本文将以策略结构中最核心的两个部分——`__init__()` 和 `next()` 为主展开，辅以其他方法，系统理解 Backtrader 策略开发。

---

## 策略类方法概览

按生命周期顺序，策略运行过程会依次触发 `bt.Strategy` 的如下方法。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-07/2025-07-06-backtrader-tutorial-p3-01-v1.png)

每个方法的含义：

- `__init__`：初始化策略所需的指标、变量和状态跟踪器，但不执行任何逻辑判断；
- `next`：每根 K 线到来时触发，是策略的主要逻辑执行区，进行买卖判断与操作；

其他四个方法：

- `start`：策略开始运行前调用一次，适合打印初始信息、初始化日志或外部资源；
- `stop`：策略运行结束时调用，用于输出最终资金、保存记录或清理环境；
- `prenext`：当数据长度不足以计算指标时调用，可用于等待指标准备就绪；
- `nextstart`：数据第一次满足指标所需长度时调用一次，标志策略正式进入主逻辑；

除了策略周期过程的几个方法，还有两个回调函数不可缺少。

* `notify_order`：订单状态变化时触发；
* `notify_trade`：交易完成时触发；

这两个系统级的回调方法能保障策略响应的及时与准确。

## 一个实际策略示例：均线+RSI 回调策略

为了更好地说明 Backtrader 策略结构，选取一个结构清晰、便于理解的示例策略：

它结合了两条均线判断趋势方向，以及 RSI 和布林带用于开仓与风控。该策略简单、信号明确，适合作为结构讲解的载体。

策略信号描述如下：

**开仓信号：** 当短期均线向上穿越长期均线，且 RSI 小于 70，表示上涨动能增强但未进入超买区域，此时尝试建立多头仓位。

**平仓信号：** 当价格上穿布林带上轨，可能处于短线过热，或价格跌破长期均线，表明趋势可能反转，此时选择平仓止盈或止损。

---

## 创建一个策略模版

在正式实现策略前，先搭建一个通用策略模板，包含策略生命周期中的关键方法结构，便于后续逐步填入具体逻辑。这有助于清晰组织策略，也方便调试扩展。

```python
class DemoStrategy(bt.Strategy):
    def __init__(self):
        print("[init] 初始化策略")

    def start(self):
        print("[start] 策略启动")

    def prenext(self):
        print(f"[prenext] 数据未准备，当前长度: {len(self)}")

    def nextstart(self):
        print("[nextstart] 数据首次满足，开始正式运行")

    def next(self):
        print("[next] 进入主逻辑判断")

    def notify_order(self, order):
        print(f"[notify_order] 状态更新: {order.getstatusname()}")

    def notify_trade(self, trade):
        print(f"[notify_trade] 交易成交")

    def stop(self):
        print("[stop] 策略结束")
```

其中包含了策略中常用方法，每个函数都加入了 print 以理解策略流程。这个模板会作为策略实现的基础结构，我们只要按需补充方法。

---

## 指标与变量初始化：`__init__`

`__init__` 方法是创建策略时最先被执行的部分，可用于定义技术指标、仓位订单、交易状态追踪器等，或是用于调试和记录的数据。

```python
def __init__(self):
    self.sma_fast = bt.indicators.SMA(period=10)
    self.sma_slow = bt.indicators.SMA(period=30)
    self.rsi = bt.indicators.RSI(period=14)
    self.bbands = bt.indicators.BollingerBands(period=20)
    
    self.order = None
    self.trade_count = 0
    self.max_drawdown = 0.0
```

要注意的是，这个阶段仅是构造指标对象，不能使用指标的值（如 `self.sma_fast[0]`），因为数据尚未推进，值将返回 `nan`。

---

## 策略核心方法：`next`

每当有新的 K 线闭合，Backtrader 就会调用 `next()`，判断信号是否满足、是否应开仓和平仓，以及实现止盈止损逻辑。

```python
def next(self):
    if len(self) < self.sma_slow.params.period:
        return  # 数据尚未准备好

    if self.order:
        return  # 如果已有订单正在处理，跳过

    # 多头信号：快线突破慢线，且 RSI 不超买
    if not self.position:
        if self.sma_fast[0] > self.sma_slow[0] and self.rsi[0] < 70:
            self.order = self.buy()

    # 止盈/止损逻辑：如果持仓，价格触及布林带上轨或跌破均线
    elif self.position:
        if self.data.close[0] > self.bbands.lines.top[0]:
            self.order = self.sell()
        elif self.data.close[0] < self.sma_slow[0]:
            self.order = self.sell()
```

毫无疑问，`next` 是策略调用最频繁和决策集中的方法，也是性能与逻辑完整性的重点部分。

---

## 多周期与多标的结构对策略的影响

在加载多个 DataFeed 时，如不同时间周期或不同交易标的，策略结构将变得更复杂。

* `next()` 会被任意数据的推进触发 → 需要判断数据来源
* 时间戳可能不同步 → 判断是否所有数据对齐
* 技术指标需绑定对应的 `data` 明确区分

以下是一个典型的双周期确认逻辑：

```python
def next(self):
    dt_1h = self.datas[0].datetime.datetime(0)
    dt_1d = self.datas[1].datetime.datetime(0)

    if dt_1h != dt_1d:
        return  # 跳过不同步的 bar

    sma_1h = bt.ind.SMA(self.datas[0], period=20)
    sma_1d = bt.ind.SMA(self.datas[1], period=20)

    if self.datas[0].close[0] > sma_1h[0] and self.datas[1].close[0] > sma_1d[0]:
        self.buy(data=self.datas[0])
```

多周期策略的重点在于**结构组织与同步判断**，不注意处理容易造成逻辑冲突或过度交易。

---

## 五、`notify_order()`：订单生命周期回调

当订单状态发生变化（提交、接受、成交、取消、拒绝）时，Backtrader 会调用该方法。你可以在这里追踪成交价、记录手续费、应对异常情况。

```python
def notify_order(self, order):
    if order.status in [order.Submitted, order.Accepted]:
        return

    if order.status in [order.Completed]:
        print(f"成交：价格 {order.executed.price}，手续费 {order.executed.comm}")
    elif order.status in [order.Canceled, order.Rejected]:
        print("订单失败：", order.getstatusname())

    self.order = None
```

在实盘环境中，这一步至关重要。

---

## 六、`notify_trade()`：交易盈亏回调

当一笔交易完成，`notify_trade` 方法会被调用。

```python
def notify_trade(self, trade):
    if trade.isclosed:
        print(f"本次交易盈亏: {trade.pnl}")
```

该函数有助于策略在运行过程中进行绩效追踪。如你可以统计盈亏、记录胜率、输出日志。

---

## 总结

Backtrader 的策略运行结构以 `__init__()` 初始化与 `next()` 执行为核心，结合数据同步、订单追踪与交易反馈，构成了完整的事件驱动系统。

理解每一个方法的职责，有助于你构建更具模块化、容错性和扩展性的交易策略。
