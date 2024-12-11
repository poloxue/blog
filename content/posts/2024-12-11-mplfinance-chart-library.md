---
title: "mplfinance 使用教程：从入门到进阶"
date: 2024-12-10T15:17:39+08:00
draft: false
comment: true
description: "mplfinance 是一个基于 matplotlib 的金融数据可视化工具，旨在方便地生成 OHLC、K线图和其他金融相关图表。本教程将逐步带你从基础到高级使用，帮助你熟练掌握 `mplfinance` 的使用技巧。"
---

`mplfinance` 是一个基于 `matplotlib` 的金融数据可视化工具，旨在方便地生成 OHLC、K线图和其他金融相关图表。

本文将带你逐步从基础到高级使用，帮助你熟练掌握 `mplfinance` 的使用技巧。

---

## 安装 `mplfinance`

在使用 `mplfinance` 前，需要先安装它。可以通过以下命令安装：

```bash
pip install --upgrade mplfinance
```

`mplfinance` 依赖于 `matplotlib` 和 `pandas`，确保你的环境中已安装这些库。

---

## 准备数据

在开始绘图之前，需要准备一个包含 OHLC 数据的 Pandas DataFrame。

我们从 Tushare 下载股票的历史数据：

```python
import pandas as pd
import datetime
import tushare as ts

pro = ts.pro_api()

data = pro.daily(ts_code="000001.SZ", start_date=start_date.strftime("%Y%m%d"))
data["trade_date"] = pd.to_datetime(data["trade_date"])
data.set_index("trade_date", inplace=True)
data.index.name = "Date"
data = data[["open", "high", "low", "close", "vol"]]
data.rename(
    columns={
        "open": "Open",
        "high": "High",
        "low": "Low",
        "close": "Close",
        "vol": "Volume",
    },
    inplace=True,
)
data.sort_index(ascending=True, inplace=True)
```

数据示例如下：

```bash
            Open  High   Low  Close      Volume
Date
2023-12-12  9.31  9.50  9.30   9.42   855016.42
2023-12-13  9.38  9.39  9.15   9.16  1061301.65
2023-12-14  9.21  9.28  9.15   9.15   742900.56
2023-12-15  9.20  9.35  9.19   9.21   988939.42
2023-12-18  9.18  9.24  9.09   9.13   654425.62
```

只保留接下来会用到的数据列。

---

## 绘制基础图表

现在，我们只要将数据导入 `mplfinance` 即可。

示例代码如下，调用 `plot` 方法，并传入从 tushare 得到的数据生成图表：

```python
import mplfinance as mpf

# 绘制基础 K 线图
mpf.plot(data, volume=True)
```

你将看到一个简单的 OHLC 图表，显示指定时间范围内的开盘、收盘、最高和最低价。参数 `volume` 设置为 `True` 表示显示成交量。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-12/2024-12-11-mplfinance-chart-library-01.png)

默认的图标类型按 ohlc 美国线风格显示，接着往下看，如何将其修改为蜡烛线。

---

### 更改图表类型

`mplfinance` 提供多种图表类型，包括默认的 `ohlc`（美国线）、`candle`（K 线图）、`line`（折线图）、`renko`（砖型图） 和 `pnf`（点数图） 等。

示例：

```python
mpf.plot(data, type='candle', volume=True)
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-12/2024-12-11-mplfinance-chart-library-02.png)

其他风格：

```python
mpf.plot(data, type='line', volume=True)
mpf.plot(data, type='renko', volume=True)
mpf.plot(data, type='pnf', volume=True)
```

---

## 添加移动平均线（MA）

mplfinance 支持了对均线的支持，可以通过 `mav` 参数添加单个或多个移动平均线。

如添加单条移动平均线，示例：

```python
mpf.plot(data, type='ohlc', mav=4)
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-12/2024-12-11-mplfinance-chart-library-03.png)

或添加多条移动平均线，示例：

```python
mpf.plot(data, type='candle', mav=(3, 6, 9))
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-12/2024-12-11-mplfinance-chart-library-04.png)

---

## 其他指标

除了均线，我们也可以在图表中叠加自己的技术指标，如 EMA，还有布林带。

可以通过 talib 计算指标，示例如下：

```python
ema20 = ta.EMA(data["Close"], timeperiod=10)
upper, middle, lower = ta.BBANDS(
    data["Close"], timeperiod=20, nbdevup=2, nbdevdn=2, matype=0
)
plots = [
    mpf.make_addplot(ema20, label="EMA10"),
    mpf.make_addplot(upper, label="BB_UPPER"),
    mpf.make_addplot(middle, label="BB_MIDDLE"),
    mpf.make_addplot(lower, label="BB_LOWER"),
]
mpf.plot(
    data,
    type="candle",
    addplot=plots,
)
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-12/2024-12-11-mplfinance-chart-library-05-v2.png)

如上的示例中，只添加了图例，除此以外，`make_addplot` 还可以设置其他属性，如 color，type 之类的。

mplfinance 的代码库中中提供了大量的代码示例，想深入了解这个库，建议多看看这些案例，访问 [examples](https://github.com/matplotlib/mplfinance/tree/master/examples)。

---

## 保存图表

绘制完成后，可以将图表保存为图片文件：

```python
mpf.plot(data, type='candle', volume=True, savefig='output.png')
```

---

## 动画与实时更新

`mplfinance` 支持动态更新图表，适合实时显示金融数据。通过 `animation` 功能，可以生成动态图表，可参考官方文档中的示例：[Animation/Updating plots](https://github.com/matplotlib/mplfinance/blob/master/markdown/animation.md)。这块就不展开介绍了，因为如果确有动态更新的需求，lightweight-charts 比 mplfinance 更合适。



---

## 总结

`mplfinance` 是一个功能强大且易于使用的金融数据可视化工具，从基础的 K 线图到高级的技术分析图表都能轻松实现。

以下是几点建议：

1. **先熟悉基础功能**：学会加载数据并绘制简单图表。
2. **逐步扩展技巧**：尝试添加移动平均线、成交量及自定义指标。
3. **参考官方教程**：多浏览示例代码和文档。

希望这篇教程能帮你快速上手 `mplfinance`！
