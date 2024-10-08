---
title: "如何通过 Python 将日线行情转为周线"
date: 2024-07-23T17:06:52+08:00
draft: false
comment: true
description: "将低周期的蜡烛图转换为长周期蜡烛图是一种常见的需求，如分钟线到小时线、日线到周线，这可以帮我们更好分析图表和产生信号。"
---

将低周期的蜡烛图转换为长周期蜡烛图是一种常见的需求，如分钟线到小时线、日线到周线，这可以帮我们更好观察和分析图表。或许，你用的行情图表软件或数据提供商都会提供多周期的数据。

今天，我将介绍如何通过 Python 实现这个需求，以日线转为周线为例。

## 基础思路

蜡烛图可以按不同时间周期，如分钟、小时、日、周和月。在转换前，先了解下蜡烛图的基本结构。

每根蜡烛表示一个时间段内的价格信息，包含四个关键数据：

- 开盘价（Open）：该时间段的第一个交易价格。
- 收盘价（Close）：该时间段的最后一个交易价格。
- 最高价（High）：该时间段内的最高交易价格。
- 最低价（Low）：该时间段内的最低交易价格。

如何将日线数据转换为周线数据？我们只要按每周的数据重新计算开盘价、收盘价、最高价和最低价即可。

规则如下：

- 开盘价（Open）：每周的开盘价是该周第一个交易日的开盘价。
- 收盘价（Close）：每周的收盘价是该周最后一个交易日的收盘价。
- 最高价（High）：每周的最高价是该周所有交易日中的最高价。
- 最低价（Low）：每周的最低价是该周所有交易日中的最低价。

有了基本思路，现在让我们来实现吧。我将通过 Python 的数据处理库 pandas 实现这个需求。

## 转换为周线数据

我直接用 tushare 下载南华期货指数的日线数据，这个数据 tushare 上没有提供周线行情。

南华连玉米指数行情下载，如下所示：

```python
import tushare as ts

pro = ts.pro_api()
df = pro.index_daily("C.NH")
```

使用 Python 的 Pandas 库即可轻松实现这一转换。

```python
import pandas as pd

# 将日期列转换为日期时间类型
df['trade_date'] = pd.to_datetime(df['trade_date'])
df.sort_values(by='trade_date', inplace=True)

# 设置日期列为索引
df.set_index('trade_date', inplace=True)

# 使用resample和agg方法将日线数据转换为周线数据
weekly_data = df.resample('W').agg({
    'open': 'first',
    'high': 'max',
    'low': 'min',
    'close': 'last'
})

print(weekly_data)
```

如上使用了`resample` 函数用于将日线数据转换为周线数据。通过设置日期索引，并使用`resample('W')`对数据重新采样，然后应用聚合函数计算每周的开盘价、收盘价、最高价和最低价，即 first、max、min 和 last。

输出：

```bash
                 open       high        low      close
trade_date
2004-09-26  1000.0000  1005.9423   988.9644  1000.8489
2004-10-03   997.4533  1005.0934   953.5251   960.9874
2004-10-10   960.3217   974.2795   948.2662   967.8320
2004-10-17   966.4796   983.7452   958.8617   963.4444
2004-10-24   966.8278   979.5159   958.3692   962.5985
...               ...        ...        ...        ...
2024-06-30  1385.0649  1416.5437  1383.9407  1410.3604
2024-07-07  1408.6740  1410.9225  1386.1892  1387.8755
2024-07-14  1387.8755  1390.6861  1350.7756  1351.3377
2024-07-21  1353.5862  1356.9589  1329.9771  1346.2786
2024-07-28  1347.9650  1356.9589  1346.2786  1352.4619

[1036 rows x 4 columns]
```

注意一点，周线的索引日期是周期的最后一天，如果不放心，可检查下最近的数据。

```python
print(df.iloc[-8:])
```

输出：

```bash
         ts_code      close       open       high        low  pre_close   change  pct_chg       vol     amount
trade_date
2024-07-11    C.NH  1356.3968  1360.3316  1364.2665  1355.2725  1360.3316  -3.9348  -0.0029  529092.0  1380710.0
2024-07-12    C.NH  1351.3377  1356.3968  1362.5801  1350.7756  1356.3968  -5.0591  -0.0037  554975.0  1394776.0
2024-07-15    C.NH  1340.0953  1353.5862  1356.9589  1335.5983  1351.3377 -11.2424  -0.0083  841183.0  1409114.0
2024-07-16    C.NH  1338.4089  1340.0953  1342.3438  1329.9771  1340.0953  -1.6864  -0.0013  626912.0  1442453.0
2024-07-17    C.NH  1342.3438  1337.2847  1345.7165  1337.2847  1338.4089   3.9349   0.0029  665443.0  1445703.0
2024-07-18    C.NH  1345.7165  1342.9059  1346.8407  1337.2847  1342.3438   3.3727   0.0025  487530.0  1456021.0
2024-07-19    C.NH  1346.2786  1347.4029  1349.0892  1339.5332  1345.7165   0.5621   0.0004  452499.0  1478614.0
2024-07-22    C.NH  1352.4619  1347.9650  1356.9589  1346.2786  1346.2786   6.1833   0.0046  545743.0  1459278.0
```

现在就顺利得到了周线数据。但还有个问题，对于长假出现的一周都没有行情数据，会出现 na 为空的数据。针对它的处理方法有两种，一种是使用 fillna 向前补齐，不过我推荐使用 dropna 删除掉这些数据即可。

```python
weekly_data.dropna(inplace=True)
```

这些周线数据可以帮助我们分析市场、产生信号，做出更明智的交易决策。

## 结论

通过 Python 数据处理库 pandas 的 resample 函数可以容易地将日线数据转换为周线。

希望这篇博文对你有用，助你更好地理解和实现不同周期数据的转化。
