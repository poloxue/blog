---
title: "Backtrader 教程一：为什么选择 Backtrader？"
date: 2025-07-03T15:18:08+08:00
draft: false
comment: true
featureimage: "https://cdn.jsdelivr.net/gh/poloxue/images@2025-07/2025-07-03-backtrader-tutorial-p1-cover.webp"
---

今天想跟大家聊一个量化交易中非常实用的工具——Backtrader。

如果你对量化交易感兴趣，可能已经听说过策略回测。简单说，就是用历史数据去验证一个交易策略有没有效。在Python里，其实有不少回测框架可以用，比如backtesting.py、vectorbt，还有已经不更新的 zipline。

但今天，我要重点介绍的是——Backtrader。

## **🤔 为什么选Backtrader？**

因为构建一个稳健的交易系统，光有策略还不够，还需要有清晰的架构、可扩展的功能，以及模块化的设计。而Backtrader，就是一个以策略回测为核心，同时帮你搭建交易系统骨架的Python框架。

它不只是回测工具，更是一个轻量级的“交易系统骨架”。

## **🔍 Backtrader是什么？**

Backtrader是一个用Python写的量化回测框架，专门为个人交易者和研究者设计。它支持策略开发、历史回测、可视化分析，甚至还能接入实盘交易。

更重要的是，它采用面向对象的设计，把策略、数据、订单、资金账户这些模块分得清清楚楚，你不用把所有代码都塞在一个脚本里，而是像搭积木一样，有条不紊地搭建系统。

## **🧱 核心架构：五大模块**

Backtrader的架构非常清晰，主要由五个模块组成：

**策略逻辑（Strategy）**  

你把买卖信号的判断写在这里，框架自动帮你执行订单，你不用操心具体怎么成交。

**数据输入（DataFeed）**  

支持从CSV、Pandas、甚至实时API导入数据，不管是日线、分钟线，还是多个股票，都能一起回测。

**交易与资金管理（Broker）**  

模拟真实市场的成交，支持市价单、限价单、滑点、佣金计算，自动更新你的账户余额和持仓。

**绩效分析（Analyzer）**  

回测结束后，自动计算收益率、最大回撤、夏普比率等指标，帮你全面评估策略表现。

**图形展示（Observer）**  

自动画出买卖点、资金曲线、技术指标，一眼看懂策略行为。

这五个模块各司其职，而你只需要专注在策略逻辑上。

## **🔄 一张图看懂回测流程**

从数据加载 → 策略判断 → 订单生成 → 成交撮合 → 绩效分析 → 图形输出，整个过程都由一个叫Cerebro的引擎统一调度。你只需要写策略，其它交给Backtrader。

## **📊 和其它框架比，Backtrader强在哪？**

✅ 完全本地运行，不依赖网络，数据更安全  
✅ 支持多策略并行运行，方便组合对比  
✅ 支持多股票、多周期数据一起分析  
✅ 自带可视化，不用另外画图  
✅ 结构清晰，组件解耦，易扩展维护  

## **💡 和手写回测的对比**

很多人一开始学量化，会用Pandas写循环来回测，比如：

```python
for i in range(1, len(data)):
    if data['close'][i] > data['close'][i-1] and position == 0:
        # 买入...
    elif ...
```

这样写虽然直接，但策略一复杂，代码就变得又乱又难维护。

而用Backtrader，同样的策略可以这样写：

```python
class MyStrategy(bt.Strategy):
    def next(self):
        if self.data.close[0] > self.data.close[-1]:
            self.buy()
        elif self.data.close[0] < self.data.close[-1]:
            self.sell()
```

清晰、干净，所有底层操作都被封装好了。

## **🎯 总结**

我喜欢Backtrader，就是因为它让我更专注于策略本身，而不是重复造轮子。

它干净、模块化、易于扩展，不管你是刚入门，还是想搭建自己的交易系统，Backtrader都是一个非常理想的起点。

下一期，我会带大家安装Backtrader，并实现一个最简单的“买入持有”策略。如果你对量化交易感兴趣，别忘了关注我，我们下期见！

