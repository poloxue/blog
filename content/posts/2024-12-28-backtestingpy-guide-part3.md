---
title: "Backtesting.py 参数优化 Part2"
date: 2024-12-24T12:15:18+08:00
draft: true
comment: true
description: "本文介绍 Backtesting.py 的参数优化，尽量把它的参数优化能力都挖掘出来。"
---


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




