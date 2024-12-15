---
title: "Backtesting.py 教程：由浅入深"
date: 2024-12-12T15:15:18+08:00
draft: true
comment: true
description: "本文将介绍 **Backtesting.py**，一个轻量级的 Python 的交易回测框架。"
---

本文将介绍 **Backtesting.py**，一个轻量级的 Python 回测框架。

## 什么是 Backtesting.py

Backtesting.py 是一个轻量级的 Python 回测框架，专注于策略回测的核心功能，适合快速实现和测试交易策略。

与其他回测框架，如 VectorBT 或 Backtrader 相比，Backtesting.py 简化了使用流程，没有过多复杂功能，自然地，它更加简单易用与高效。

Backtesting.py 内置了策略参数优化工具，可与 Pandas 数据框和 NumPy 数组兼容，易于集成。当然简单是有代价的，它的缺点是不支持多资产交易和分数股交易，这也限制了它的普适性。

注：策略回测并不一定需要专门的回测框架，也可以完全通过像 NumPy 和 Pandas 这样的向量化计算库来实现。

## 快速开始

让我们快速上手 Backtesting.py 的使用吧。

### 安装

确保你已经有了 Python 环境，并成功安装 backtesting.py 和 TA-Lib 两个依赖包，
安装命令如下：

```bash
pip install backtesting TA-Lib
```

如果 TA-Lib 安装遇到困难，请查看 [TA-Lib 安装指南](https://ta-lib.github.io/ta-lib-python/install.html)，或者也可以选择其他技术指标库，如 pandas-ta。

### 编写第一个策略

接下来，通过 backtesting.py 回测将实现一个最简单的均线交叉策略。策略的规则也很简单，就是 SMA 金叉买入、死叉卖出。

**导入必要的模块**

首先，创建一个名为 `strategy.py` 的 Python 文件，开始编写我们的策略。

导入要用到的模块：

```python
from backtesting import Backtest, Strategy
from backtesting.test import GOOG  # 示例数据
import talib
```

**定义策略类**

定义一个名为 `MovingAverageCrossStrategy` 策略类，继承 `Strategy`：

```python
class MovingAverageCrossStrategy(Strategy):
    
    # 定义参数
    fast_ma_window = 10
    slow_ma_window = 20

    def init(self):
        self.fast_ma = self.I(talib.SMA, self.data.Close, timeperiod=self.fast_ma_window)
        self.slow_ma = self.I(talib.SMA, self.data.Close, timeperiod=self.slow_ma_window)

   def next(self):
       if self.rsi[-1] > 70:
           self.position.close()
       elif self.rsi[-1] < 30:
           self.buy()
```

**运行回测**

使用内置的 Google 数据运行回测：
```python
bt = Backtest(GOOG, RSIStrategy, cash=10000, commission=0.002)
stats = bt.run()
print(stats)
bt.plot()
   ```

### 结果解读

运行上述代码后，你将看到：

- **策略统计数据**：包括初始资金、最终净值、最大回撤等。
- **交互式回测图表**：图表显示了买卖点、指标和交易统计信息。

在短短 30 行代码内，我们实现了一个简单的 RSI 交叉策略。这是 Backtesting.py 简洁高效的一个典型例子。

## 参数优化

在策略开发中，参数优化是提升回测表现的重要步骤。Backtesting.py 内置了参数优化功能，能够帮助开发者高效找到最佳策略参数组合。

### 使用优化器

以下是一个优化 RSI 策略的示例代码：
```python
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

# 运行优化
bt = Backtest(GOOG, RSIStrategy, cash=10000, commission=0.002)

stats = bt.optimize(
    upper_bound=range(50, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=range(10, 30, 2),
    maximize='Sharpe Ratio',
    constraint=lambda param: param.lower_bound < param.upper_bound
)
print(stats)
bt.plot()
```

### 优化器参数说明

- **`upper_bound` 和 `lower_bound`**：分别表示 RSI 的超买和超卖阈值。
- **`rsi_window`**：RSI 的时间窗口大小。
- **`maximize`**：优化的目标，可以选择 `Sharpe Ratio`、`Equity Final [$]` 等指标。
- **`constraint`**：添加约束条件，确保参数组合的合理性。

### 并行计算和性能优化

Backtesting.py 支持多线程并行计算，可以充分利用计算机的多核性能：

```python
import os
os.environ['BACKTESTING_NUM_WORKERS'] = '4'  # 使用 4 个线程
```

在运行优化时，Backtesting.py 会并行计算每种参数组合，显著提高效率。

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

## 多时间框架策略

在某些策略中，我们希望结合多时间框架的数据来提高交易决策的可靠性。例如，除了每日 RSI（Relative Strength Index），我们还可以添加每周 RSI，并要求在两个时间框架的指标满足条件时才进行交易。

### 实现多时间框架策略

以下是如何在策略中实现多时间框架的步骤：

1. **导入所需函数**：
```python
from backtesting.lib import resample_apply
```

2. **定义每周 RSI**：
在策略的 `__init__` 方法中添加每周 RSI 的计算：
```python
self.daily_RSI = self.I(TA.RSI, self.data.Close, self.rsi_window)
self.weekly_RSI = resample_apply('W', TA.RSI, self.data.Close, self.rsi_window)
```
这里使用 `resample_apply` 函数，将每日数据降采样为每周数据，并计算 RSI。

3. **修改交易逻辑**：
在 `next` 方法中，结合每日和每周 RSI 来决定交易：
```python
if crossover(self.daily_RSI, self.upper_bound) and self.weekly_RSI[-1] > self.upper_bound:
    self.position.close()

if crossover(self.lower_bound, self.daily_RSI) and self.weekly_RSI[-1] < self.lower_bound:
    self.buy()
```
此逻辑要求每日 RSI 与阈值交叉，并且每周 RSI 同时满足条件。

### 代码示例
以下是完整的多时间框架策略示例：
```python
from backtesting.lib import resample_apply

class MultiTimeFrameRSIStrategy(Strategy):
    rsi_window = 14
    upper_bound = 70
    lower_bound = 30

    def init(self):
        self.daily_RSI = self.I(TA.RSI, self.data.Close, self.rsi_window)
        self.weekly_RSI = resample_apply('W', TA.RSI, self.data.Close, self.rsi_window)

    def next(self):
        if crossover(self.daily_RSI, self.upper_bound) and self.weekly_RSI[-1] > self.upper_bound:
            self.position.close()

        if crossover(self.lower_bound, self.daily_RSI) and self.weekly_RSI[-1] < self.lower_bound:
            self.buy()
```

### 策略优化与扩展

- **参数优化**：
  可以将 `weekly_RSI` 的窗口期设为一个独立参数，并使用优化器来测试最佳值。

- **更多条件**：
  添加更多条件，例如检查每日 RSI 是否持续下降，以增强策略的灵活性。

通过结合多时间框架数据，我们可以更好地过滤噪声，提高交易决策的准确性。

## 做空、止损和止盈

除了基本的多头交易策略，我们还可以在 Backtesting.py 中实现做空、止损和止盈功能。

### 实现做空
在 RSI 策略的基础上，可以通过以下代码实现做空：
```python
def next(self):
    if crossover(self.daily_RSI, self.upper_bound):
        if self.position.is_long:
            self.position.close()
        self.sell()

    if crossover(self.lower_bound, self.daily_RSI):
        if self.position.is_short:
            self.position.close()
        self.buy()
```
此代码实现了 RSI 超过上限时平多头并开空头，而 RSI 低于下限时平空头并开多头。

### 止损和止盈

通过设置止损（Stop Loss, SL）和止盈（Take Profit, TP），可以更好地控制风险和锁定收益：
```python
def next(self):
    price = self.data.Close[-1]
    sl = 0.95 * price  # 止损设为当前价格的 95%
    tp = 1.15 * price  # 止盈设为当前价格的 115%

    if crossover(self.lower_bound, self.daily_RSI):
        self.buy(sl=sl, tp=tp)

    if crossover(self.daily_RSI, self.upper_bound):
        self.sell(sl=sl, tp=tp)
```
此代码在每次开仓时，同时设置止损和止盈价格。

## 订单大小

在策略中，订单大小的控制至关重要，它直接影响到风险管理和资金利用效率。Backtesting.py 提供了多种设置订单大小的方法，以满足不同交易策略的需求。

### 使用资金比例

可以通过设置 `size` 参数为小数值，指定每笔交易占用可用资金的比例。例如：
```python
self.buy(size=0.1)
```
此代码表示每次买入使用可用资金的 10%。

#### 示例
以下是一个使用资金比例的完整示例：
```python
if crossover(self.lower_bound, self.daily_RSI):
    self.buy(size=0.1)
```
在上述示例中，每次 RSI 低于下限时，将使用当前可用资金的 10% 进行买入。

**注意事项**：
- 默认情况下，Backtesting.py 不支持部分股票买卖，仅支持整数股。因此，交易规模可能会向下取整。

### 使用固定数量

除了使用资金比例，还可以直接指定买入或卖出的数量。例如：
```python
self.buy(size=1)
```
此代码表示每次买入 1 股。

#### 示例
以下是一个按固定数量交易的完整示例：
```python
if crossover(self.lower_bound, self.daily_RSI):
    self.buy(size=1)
```
每次 RSI 低于下限时，将买入 1 股。

### 动态调整订单大小

订单大小可以基于策略动态调整。例如，可以在策略初始化时设置一个变量，并在交易逻辑中引用：
```python
class DynamicSizeStrategy(Strategy):
    def init(self):
        self.position_size = 1

    def next(self):
        if crossover(self.lower_bound, self.daily_RSI):
            self.buy(size=self.position_size)
```
还可以使用优化器来测试不同的订单大小：
```python
bt.optimize(position_size=range(1, 10))
```

### 与其他条件结合

订单大小可以与其他条件结合，例如动态计算当前价格低于指定阈值的天数：
```python
if bars_since(self.daily_RSI > self.upper_bound) >= 5:
    self.buy(size=2)
```
此逻辑表示如果 RSI 超过上限已经过去了至少 5 天，则买入 2 股。

### 应用场景

- **资金管理**：通过比例买入控制资金利用率，降低单笔交易的风险。
- **网格交易**：在特定价格区间内分批买入或卖出。
- **优化策略**：通过优化订单大小，找到最优交易规模。

通过灵活设置订单大小，可以更好地满足策略的需求，同时提高资金的利用效率和交易的灵活性。

## 交易记录导出

在策略回测完成后，我们通常希望能够导出交易记录，便于进一步分析或保存。

### 提取交易记录

Backtesting.py 会在回测结果中生成一个包含所有交易详情的 Pandas 数据框，可以通过以下方式提取：
```python
trades = stats['_trades']
print(trades.to_string())
```

### 交易记录内容

交易记录包括以下信息：
- **Entry/Exit**：交易的开仓和平仓时间。
- **Size**：交易的头寸大小。
- **P/L**：交易的利润或亏损。
- **Duration**：交易的持续时间。

这些数据可以方便地用于后续分析，例如计算平均持仓时间、每笔交易的盈亏比等。

### 导出到文件

如果需要保存交易记录，可以将其导出为 CSV 文件：
```python
trades.to_csv('trades.csv')
```
或者保存为 Excel 文件：
```python
trades.to_excel('trades.xlsx')
```

### 可视化交易记录

导出的交易记录可以使用工具（如 Excel 或 Python 的可视化库）进行分析。例如，使用 Matplotlib 绘制每笔交易的盈亏分布：
```python
import matplotlib.pyplot as plt

trades['P/L'].hist(bins=20)
plt.title('Profit/Loss Distribution')
plt.xlabel('Profit/Loss')
plt.ylabel('Frequency')
plt.show()
```
此代码生成一个柱状图，展示交易盈亏的分布情况。

通过灵活导出和处理交易记录，可以更全面地分析策略表现并进行优化。

## BarsSince 函数

在 Backtesting.py 中，有一个非常实用的函数名为 `bars_since`，可以用来简化某些条件逻辑。这个函数能够返回某个条件最后一次为真的时间距离当前有多少个 bar（时间周期），对于实现复杂的交易逻辑非常有帮助。

### BarsSince 函数的功能

`bars_since` 的主要功能是：
- 给定一个返回布尔值的条件。
- 返回距离该条件最后一次为真的时间间隔（以 bar 为单位）。

例如，如果我们希望一个条件满足后，等待三天再进行交易，这个函数可以帮助我们轻松实现，而无需手动编写多个嵌套的逻辑判断。

### 基础用法
以下是一个使用 `bars_since` 的示例，结合 RSI 指标实现更复杂的交易逻辑：

```python
from backtesting.lib import bars_since

def next(self):
    # 条件：RSI 是否在上限以下
    condition = self.daily_RSI < self.upper_bound

    # 计算距离条件最后一次满足的时间间隔
    bars = bars_since(condition)

    # 在满足条件后等待 3 个 bar 再进行交易
    if bars == 3:
        self.sell()
```
在这个示例中，`bars_since(condition)` 会返回上一次 RSI 低于上限的时间距离。

### 条件组合

可以结合多个条件使用 `bars_since`，例如：

```python
from backtesting.lib import bars_since

def next(self):
    # 条件：RSI 高于上限且价格高于均线
    condition = (self.daily_RSI > self.upper_bound) & (self.data.Close > self.data.Close.rolling(20).mean())

    # 检查条件满足后的时间间隔
    bars = bars_since(condition)

    if bars == 5:
        self.buy()
```
此示例中，交易逻辑在条件满足 5 天后执行。

### 示例：RSI 持续超买条件

假设我们希望在 RSI 连续超买 3 天后卖出，可以使用以下代码实现：

```python
def next(self):
    if bars_since(self.daily_RSI < self.upper_bound) == 3:
        self.sell()
```
此逻辑会在 RSI 跌破上限后持续超买 3 天时触发卖出操作。

### 注意事项

- `bars_since` 返回的值是整数。
- 如果条件从未满足过，`bars_since` 会返回无穷大（`float('inf')`）。在编写策略时需特别注意这种情况，避免因无穷大值导致的逻辑错误。

### 优势

使用 `bars_since` 函数可以显著减少代码的复杂性，尤其是在处理多条件和时间间隔相关的逻辑时。通过简化判断条件的书写，可以更高效地实现复杂策略。

`bars_since` 是 Backtesting.py 中一个强大且灵活的工具，它可以显著简化基于时间间隔的条件逻辑。无论是简单的 RSI 条件还是复杂的多条件组合，这个函数都可以轻松应对。在策略开发过程中，合理使用 `bars_since` 可以让代码更简洁、逻辑更清晰，为策略优化和扩展打下良好的基础。


