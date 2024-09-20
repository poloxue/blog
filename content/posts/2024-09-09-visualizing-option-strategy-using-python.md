---
title: "Python 三方库 Opstrat 绘制收益图加深期权策略理解"
date: 2024-09-09T11:43:05+08:00
draft: false
comment: true
description: "本文介绍介绍一个 Python 包- opstrat，通过它绘制期权收益图，帮助我们理解期权策略的收益风险比。"
---

本文介绍介绍一个 Python 包- opstrat，通过它绘制期权收益图，帮助我们理解期权策略的收益风险比。

## 什么是期权收益图？

期权收益图展示了某个期权或期权组合的盈亏情况，如下图中，展示一张标的现价 100，行权价 102 的看涨期权合约。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-09/2024-09-09-visualizing-option-strategy-02.png)

这种图表是理解风险与收益的重要工具，尤其是在期权的交易结构中，通过这些图表可以直观地看到交易的潜在利润与风险区间。

## Opstrat

[Opstrat](https://github.com/hashABCD/opstrat) 是一个 Python 工具包，它提供了便捷的方法来可视化期权交易策略，无需复杂的编程知识。通过这个工具，交易者能够快速构建单个期权或者多个期权组合的收益图。

## 安装

你可以通过以下命令安装 Opstrat：

```bash
pip install opstrat
```

该包会自动安装所需的依赖库，包括 `pandas`、`matplotlib`、`seaborn` 和 `yfinance`，这些都是用来处理数据和绘图的库。

## 使用 Opstrat 进行期权可视化

安装完成后，你可以通过如下方式导入该包：

```python
import opstrat as op
```

接下来，我们将展示如何使用 `opstrat` 绘制各种期权策略的收益图。

### 绘制单个期权的收益图

要绘制单个期权的收益图，你可以使用 `single_plotter()` 函数。

如绘制一张默认的看涨期权收益图：

```python
op.single_plotter()
```

默认生成一个看涨期权的收益图，现价为 100 元，行权价为 102 元。

如下图所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-09/2024-09-09-visualizing-option-strategy-02.png)

### 自定义单腿期权

通过提供参数，你可以自定义这个图表。例如，假设交易者卖出了行权价为 400 元的看跌期权，并收取了 12 元的期权费。以下代码将生成该卖出期权的收益图：

```python
op.single_plotter(spot=400, strike=400, op_type='p', tr_type='s', op_pr=12)
```

如下图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-09/2024-09-09-visualizing-option-strategy-03.png)

这个图表显示不同价格水平下的潜在盈利或亏损。`tr_type` 参数 `s` 表示 `short` 空这张 PUT 期权，如果为 `b` 表示是 `buy`。

### 构建多腿期权组合

如果想要构建多个期权头寸的组合策略，例如卖出跨式或铁鹰策略，可以使用 `multi_plotter()` 函数。

#### 示例 1：卖出跨式策略

卖出跨式策略包括卖出略微虚值的看涨期权和看跌期权。

在这个例子中，假设基础资产的现价为 100 元，交易者卖出了行权价分别为 110 元和 95 元的看涨期权和看跌期权。

```python
op_1 = {'op_type':'c','strike':110,'tr_type':'s','op_pr':2}
op_2 = {'op_type':'p','strike':95,'tr_type':'s','op_pr':6}
op.multi_plotter(spot=100, op_list=[op_1,op_2])
```

如下图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-09/2024-09-09-visualizing-option-strategy-04.png)

图中即展示了单腿，也展示了组合的收益曲线。

#### 示例 2：铁鹰策略

更复杂的策略，如铁鹰策略，它结合了两个看涨期权和两个看跌期权。

假设某只股票当前价格为 5000 元，交易者卖出行权价为 5100 元的看涨期权，同时买入行权价为 5200 元的看涨期权；同时，卖出行权价为 4900 元的看跌期权，买入行权价为 4800 元的看跌期权。

```python
op1={'op_type': 'c', 'strike': 5100, 'tr_type': 's', 'op_pr': 120}
op2={'op_type': 'c', 'strike': 5200, 'tr_type': 'b', 'op_pr': 80}
op3={'op_type': 'p', 'strike': 4900, 'tr_type': 's', 'op_pr': 150}
op4={'op_type': 'p', 'strike': 4800, 'tr_type': 'b', 'op_pr': 100}

op_list=[op1, op2, op3, op4]
op.multi_plotter(spot=5000,spot_range=10, op_list=op_list)
```
如下图所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-09/2024-09-09-visualizing-option-strategy-05-v1.png)

这个策略适用于对基础资产价格波动不大的市场，在有限的风险下捕捉有限的利润。

## 最后

通过 Opstrat 工具包，我们可以快速直观地构建各种期权策略的收益图，无论是简单的单腿期权，还是复杂的多腿期权组合。这提高了我们的分析效率，助我们理解期权交易的风险与收益。


