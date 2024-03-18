---
title: "每周策略：宽基指数-双均线策略"
date: 2024-12-30T22:55:27+08:00
draft: true
comment: true
description: ""
---

本文介绍使用 Python 回测双均线策略。

- 前言
  - 什么是 "宽基指数"？
    - 是指覆盖面广泛、具有相当代表性的指数基金。
    - 代表性指数，如上证综指、沪深300指数、创业板指、中证 500 指数等等。
    - 宽指特点，成分股较多，每只股票的权重较低。
  - 什么是 "均线"？
    - 均线，又称移动平均线，它是以道琼斯的 "平均成本概念" 为理论基础。
    - 它采用统计学中 "移动平均"，将一段时间内的价格平均并连成一条线。
    - 它可用来显示股票价格的历史波动情况，是反应价格运行趋势的重要指标。
    - 公式：
- 策略
  - 策略指标：
    - 短期均线，或称快钱；
    - 长期均线，或称慢线；
  - 交易信号：
    - 快均线上穿慢线；
- 准备
  - 依赖下载：pip install talib tushare  backtesting 
  - 数据处理：
    - 数据来源，假设选自于 tushare；
    - 实现一个工具函数 history_data，将数据处理处理为可用于 backtesting.py 的数据；
- 规则
  - 交易：
    - 开仓信号：短期均线上穿长期均线，开仓买入；
    - 平仓信号：短期均线下穿长期均线，平仓卖出；
  - 其他：
    - 止损模块：暂无；
    - 止盈模块：暂无；
- 代码：
  - 导入依赖包
    - import talib as ta
    - from backtesting import Backtest, Strategy
    - from data import history_data
  - 策略类定义
    - class MovingAverageStrategy(Strategy):
  - 指标计算
    - self.I(ta.SMA, self.data.Close, self.fast_period)
    - self.I(ta.SMA, self.data.Close, self.slow_period)
  - 回测函数
    - 计算信号
      - is_long_entry = last_fast_ma > last_slow_ma and not self.position
      - is_long_exit = last_fast_ma < last_slow_ma and self.position
    - 执行交易：
      - if is_long_entry: self.buy()
      - if is_long_entry: self.position.close()
- 回测策略：
  - 配置：bt = Backtest(data, MovingAverageStrategy, cash=1e8, commission=0.001)
  - 运行：bt.run()
  - 报告：bt.plot()
- 策略优化：
  - 优化：bt.optimize 替换 bt.run()
    - 算法：网格遍历算法；
    - 参数：
      - fast_period 从 5 至 30，步幅为 5；
      - fast_period 从 30 至 120，步幅为 5；
    - bt.optimize(fast_period=range(5, 30, 5), slow_period=range(30, 120, 5))
- 策略评价
  - 评价指标：
    - 收益率：
    - 回撤：
    - 夏普比率：
    - 胜率：
  - 逻辑思维：
    - 简单唯美；

- 参考
  - [A股的筋络！均线指标究竟有何奥秘？](https://www.21jingji.com/article/20230123/herald/d8f87754436b8418270afc3bc7e1ed44.html#:~:text=%E6%89%80%E8%B0%93%E5%9D%87%E7%BA%BF%EF%BC%8C%E5%8F%88%E5%8F%AB%E7%A7%BB%E5%8A%A8,%E4%BB%B7%E6%A0%BC%E7%9A%84%E5%8E%86%E5%8F%B2%E6%B3%A2%E5%8A%A8%E6%83%85%E5%86%B5%E3%80%82)
