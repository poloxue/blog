---
title: "Backtesting.py 教程：由浅入深"
date: 2024-12-12T15:15:18+08:00
draft: true
comment: true
description: "本文将介绍 **Backtesting.py**，一个轻量级的 Python 的交易回测框架。"
---

## 自定义优化指标

Backtesting.py 提供了自定义优化指标的功能，使用户可以优化符合自身需求的特定指标。

### 定义自定义优化函数

以下是一个优化策略的示例代码，基于“在市场中停留最少时间获取最大收益”的自定义指标：

```python
def my_optim_func(series):
    final_equity = series['Equity Final [$]']
    exposure_time = series['Exposure Time [%]']
    return final_equity / exposure_time

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

# 使用自定义优化函数
bt = Backtest(GOOG, RSIStrategy, cash=10000, commission=0.002)

stats = bt.optimize(
    upper_bound=range(50, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=range(10, 30, 2),
    maximize=my_optim_func,
    constraint=lambda param: param.lower_bound < param.upper_bound
)
print(stats)
bt.plot()
```

### 添加条件约束

通过为优化器添加条件约束，可以确保参数范围的合理性。例如，要求交易次数不低于 10 次：

```python
def my_optim_func(series):
    if series['# Trades'] < 10:
        return -1  # 返回负值以避免选择
    final_equity = series['Equity Final [$]']
    exposure_time = series['Exposure Time [%]']
    return final_equity / exposure_time
```

通过自定义优化指标，你可以更灵活地定义回测目标并发现符合特定需求的最佳参数组合。

## 回测结果保存

Backtesting.py 提供了保存回测结果的功能，便于后续分析。

### 保存 HTML 文件

运行回测时，Backtesting.py 会自动生成一个 HTML 文件，包含交互式图表和详细的回测结果。

```python
bt.plot(filename='results/plot.html')
```

在上述代码中，回测结果将保存在 `results` 文件夹下。如果文件夹不存在，需要提前创建：

```bash
mkdir -p results
```

### 动态命名文件

可以动态设置文件名以区分不同的参数组合。例如：

```python
lower_bound = 30
upper_bound = 70
bt.plot(filename=f'results/plot_lb{lower_bound}_ub{upper_bound}.html')
```

这将保存文件名为 `plot_lb30_ub70.html` 的 HTML 文件，方便管理不同的回测结果。

### 提取回测参数

可以通过 `stats` 对象提取回测参数，用于动态命名文件：

```python
lower_bound = stats._strategy.lower_bound
upper_bound = stats._strategy.upper_bound
bt.plot(filename=f'results/plot_lb{lower_bound}_ub{upper_bound}.html')
```

通过保存 HTML 文件和动态命名功能，你可以轻松管理和归档不同的回测结果，为后续优化和分析提供便利支持。

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


