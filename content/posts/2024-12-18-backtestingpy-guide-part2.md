---
title: "Backtesting.py 参数优化 Part1"
date: 2024-12-24T12:15:18+08:00
draft: true
comment: true
description: "本文介绍 Backtesting.py 的参数优化，尽量把它的参数优化能力都挖掘出来。"
---

上篇文章介绍了如何使用 **Backetsting.py** 快速上手。今天继续介绍另一个策略回测时非常重要的点，参数优化。参数优化是提升策略表现的重要步骤。**Backtesting.py** 内置了参数优化功能，使用起来还是很方便的。演示案例还是用上篇文章中的均线交叉策略。

## 优化示例

**Backtesting.py** 默认优化方式是将是将所有的参数组合穷举遍历一遍，这种方式也被称为-**网格搜索**。**网格搜索**适用于参数空间较小的情况；对于超参数的场景，还要考虑其他优化方法。我还是先介绍这个默认的优化方法。

以下是优化 SMACross 策略的示例代码：

```python
bt = Backtest(GOOG, SMACross, cash=10000, commission=0.002)
stats = bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
)
print(stats)
bt.plot()
```

如上通过 `range` 指定了 `fast_ma_window` 和 `slow_ma_window` 的参数范围，运行的结果就是表现最好的参数的结果。

我们可以通过 `stats._strategy` 拿到策略实例，然后打印策略实例参数。

```python
print(stats._strategy)
print("fast_ma_window", stats._strategy.fast_ma_window)
print("slow_ma_window", stats._strategy.slow_ma_window)
```

输出：

```bash
SMACrossStrategy(fast_ma_window=5,slow_ma_window=20)
fast_ma_window 5
slow_ma_window 20
```

## 参数约束

上面的优化代码有个明显的不合理，可能出现 `fast_ma_window` 小于 `slow_ma_window` 的情况，如 `fast_ma_window=20`，但 `slow_ma_window=10` 的组合，这不仅会增加优化耗时，也不符合策略的逻辑，。`backtestingpy` 的优化器提供了参数约束能力，我们可以限制 `fast_ma_window` 必须小于 `slow_ma_window`。

示例代码：

```python
stats = bt.optimize(
    fast_ma_window=range(5, 30, 5),
    slow_ma_window=range(10, 60, 10),
    constraint=lambda p: p.fast_ma_window < p.slow_ma_window,
)
```

通过 `constraint` 指定参数约束函数，限制 `fast_ma_window` 必须小于 `slow_ma_window`。这能极大提升回测优化速度。

## 评价标准

优化器默认只返回了一个表现最好的参数，但这个最好是如何评价的呢？

backtestingpy 优化器的默认评价标准是 SQN，它的计算公式如下：

```
SQN = (平均每笔交易收益 × √交易次数) / 每笔交易收益的标准差
```

一句话表述它的含义就是，SQN 通过平均收益（高更好）、收益波动性（低更好）和交易次数（多更好）衡量策略稳健性，值越高越优质。



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

通过匿名函数 `maximize=lambda r: -r["Volatility (Ann.) [%]"]` 选择年化波动波动率最小的组合。

不过，从我平时的使用体验来看，还是默认的 SQN 的优化结果比较合理，它的评价维度更加全面。

## 自定义评价指标

除了那些内置的指标，我们也可以自定义优化指标。举个例子，我们现在想寻找 "在市场中持仓时间最短收益最大” 的参数组合。

定义优化目标函数：

```python
def equity_per_exposure(r):
    final_equity = r['Equity Final [$]']
    exposure_time = r['Exposure Time [%]']
    return final_equity / exposure_time
```

目标函数是通过最终净值除以仓位持有时间计算得到，将其传递给 `bt.optimize` 即可。
```python
stats = bt.optimize(
    maximize=equity_per_exposure,
    constraint=lambda p: p.fast_ma_window < p.slow_ma_window.
)
```

如果你使用通过为优化器添加条件约束，可以确保参数范围的合理性。例如，要求交易次数不低于 10 次：

```python
def my_optim_func(series):
    if series['# Trades'] < 10:
        return -1  # 返回负值以避免选择
    final_equity = series['Equity Final [$]']
    exposure_time = series['Exposure Time [%]']
    return final_equity / exposure_time
```

通过自定义优化指标，你可以更灵活地定义回测目标并发现符合特定需求的最佳参数组合。


## 随机网格搜索

在优化过程中，测试所有参数组合可能非常耗时。例如，当存在 20,000 种参数组合时，逐一测试显然不现实。为此，Backtesting.py 提供了随机网格搜索功能，可以显著减少计算量。

### 设置最大尝试次数

通过设置最大尝试次数，可以限制测试的组合数量，实现随机网格搜索：

```python
stats = bt.optimize(
    upper_bound=range(50, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=range(10, 30, 2),
    maximize='Sharpe Ratio',
    constraint=lambda param: param.lower_bound < param.upper_bound,
    max_tries=100
)
```

上述代码将限制优化器仅测试 100 个随机选择的参数组合，而不是测试所有可能的组合。这种方法可以在以下情况下使用：

1. **快速验证策略**：在开发阶段快速检查策略表现。
2. **减少过拟合风险**：通过随机选择参数组合，降低找到过拟合结果的可能性。

### 效果演示

运行多次随机网格搜索后，你会发现结果各不相同。这是因为每次运行时，优化器随机选择不同的参数组合：

```python
for _ in range(3):
    stats = bt.optimize(
        upper_bound=range(50, 85, 5),
        lower_bound=range(10, 45, 5),
        rsi_window=range(10, 30, 2),
        maximize='Sharpe Ratio',
        max_tries=100
    )
    print(stats)
```

上述代码将在终端输出不同的优化结果，从中可以分析出随机网格搜索的有效性。


## 参数优化热力图

参数优化热力图是一种直观的方式来比较不同参数组合对策略表现的影响。通过可视化参数范围内的结果，可以快速识别最佳参数区域。

### 生成基础热力图

以下是生成基础热力图的步骤：

1. 在优化中启用热力图选项：
```python
stats, heatmap = bt.optimize(
    upper_bound=range(50, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=14,  # 固定 RSI 时间窗口
    maximize='Sharpe Ratio',
    return_heatmap=True
)
```
此代码会返回 `heatmap` 对象，其中包含优化的所有组合结果。

2. 使用 Pandas 对数据进行处理：
```python
hm = heatmap.groupby(['upper_bound', 'lower_bound']).mean()
print(hm)
```
`groupby` 方法会将数据按 `upper_bound` 和 `lower_bound` 分组，并计算每组的均值。结果是一个整齐的二维表格，适合绘制热力图。

3. 使用 Seaborn 绘制热力图：
```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.heatmap(hm.unstack(), cmap='viridis')
plt.show()
```
此代码生成了一个使用 `viridis` 配色方案的热力图，直观展示了 `upper_bound` 和 `lower_bound` 对 Sharpe Ratio 的影响。

### 自定义热力图配色

Seaborn 提供了丰富的配色方案，可以根据需求调整热力图的外观。例如：
```python
sns.heatmap(hm.unstack(), cmap='plasma')
plt.show()
```
可以通过更改 `cmap` 参数选择不同的颜色方案，如 `plasma`、`coolwarm` 或 `inferno`。

### 多参数组合的热力图

当涉及三个或更多参数时，可以生成所有可能组合的二维热力图。Backtesting.py 提供了内置方法：

1. 导入工具函数：
```python
from backtesting.lib import plot_heatmaps
```

2. 调用 `plot_heatmaps` 绘制多个热力图：
```python
plot_heatmaps(heatmap, aggfunc='mean')
```
`aggfunc` 参数指定聚合方式，默认为 `mean`，即对第三个参数取平均值。

此函数会生成多个热力图，分别展示各参数对的关系。例如：
- `upper_bound` 和 `lower_bound`
- `upper_bound` 和 `rsi_window`
- `lower_bound` 和 `rsi_window`

### 注意事项

- 热力图适用于分析两两参数之间的关系，当参数过多时，图表数量会急剧增加。
- 使用合理的参数范围和步长可以减少计算量并提高可读性。

通过参数优化热力图，可以高效地探索参数空间，找到表现最佳的策略组合。


