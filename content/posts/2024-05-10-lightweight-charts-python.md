---
title: "一个 Python 轻量级交易图表库 - lightweight-charts-python"
date: 2024-05-13T16:23:09+08:00
draft: false
comment: true
description: "这两天发现一个可在 Python 显示交易图表的库，名为 lightweight-charts-python。顾名思义，它是基于 tradingview 轻量级库 lightweight-charts 开发而来。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-01.png)

今天这篇可能有点跨域了，其实这段时间，我一直在做交易。如果大家对自动化交易感兴趣，我也有很多内容可以写。

这两天发现一个可在 Python 显示交易图表的库，名为 lightweight-charts-python。顾名思义，它是基于 tradingview 轻量级库 lightweight-charts 开发而来。

TradingView 是一款非常流行的交易行情分析软件，而 lightweight-charts 是它提供的精简版 Js 开源库。因为 TradingView 本身就是一款专注交易的软件，lightweight-charts 有这高性能和用户体验友好的特点。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-02.png)

它的实现是 Python 通过 webview 集成 lightweight-charts 开发而来的库，使用 Python 的小伙伴无需学习 Javascript 也能使用 lightweight-charts 了。

通过 Python 实现，还有个优势，容易与其他框架结合，如国内非常流行的实盘框架 vnpy，它的 GUI 界面就是用 pyside6 实现，这个库非常容易集成到 pyside6 中，从而集成到 vnpy。

OK，让我们正式开始演示这个库的使用吧。 

## 安装命令

```bash
pip install lightweight-charts
```

## 行情数据

为了体验 lightweight-charts-python 库的使用，提前准备行情数据必要的步骤。

lightweight-charts-python 的 GitHub 仓库中提供了一些样例数据，我们也可以从一些交易软件下载。不过，既然都来看这篇文章了，肯定都是会编程的，我推荐开源库获取数据。

我将用 tushare 下载行情数据，或者也可以用 akshare 等一些开源库。假设下载代码为 000001.SZ 的平安银行。

```python
import tushare as ts

pro = ts.pro_api()
code = "000001.SZ"
df = pro.query("daily", ts_code=code, start_date="20180701", end_date="20240510")
```

通过 pandas 将其整理为满足 lightweight-charts 目标的数据格式，即包含 date、open、high、low、close、volume 字段，还要保证 date 为日期格式，且数据按日期升序排列。

```python
df["date"] = pd.to_datetime(df["trade_date"])
df = df.sort_values(by="date", ascending=True)
df.rename(columns={"vol": "volume"}, inplace=True)
df = df[["date", "open", "high", "low", "close", "volume"]]

df.to_csv(f"D_{code}.csv")
```

目录就有了名为 D_000001.SZ.csv 的行情数据文件。依次类推，还可下载周线和月线。分钟线数据好像需要更高积分才能下载。

## 快速上手

首先，体验下它最基础使用方法，显示历史 K 线图表。

```python
import time
import pandas as pd

from lightweight_charts import Chart

def main():
  chart = Chart()

  df = pd.read_csv("D_000001.SZ.csv")
  chart.set(df)

  chart.show(block=True)

if __name__ == "__main__":
  main()
```

如上的代码中，`Chart` 是它的核心类，通过 pandas 的读取 csv 文件数据并将其 DataFrame 传递给 chart 即可完成历史行情的设置。

运行代码，你将能看到如下的效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-03.png)

## 新增 Bar/Tick

如要实时的行情，我们可通过 `Chart` 提供的方法更新图表。它提供了两个方法，分别是 `update` 可用于更新 Bar 蜡烛图和 `update_from_tick` 从 tick 数据更新图表。

首先，演示新增 bar 更新图表。我们将之前准备的数据截断，`df.iloc[-10:]` 保留 10 个 bar 为演示数据。

```python
chart = Chart()

df = pd.read_csv("D_000001.SZ.csv")
chart.set(df.iloc[:-10])

chart.show()

for i, bar in df.iloc[-10:].iterrows():
    chart.update(bar)
    time.sleep(1)
```

每隔 1 秒，新增 1 个 bar。动图效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-04.gif)

实际场景下，实时更新 tick 才是我们的目标。只要将实时 tick 不断投喂给 chart 即可。我暂时没有接入实时数据，将用随机模拟的方式生成后续数据。

```python
chart = Chart()

df = pd.read_csv("D_000001.SZ.csv", parse_dates=["date"])
chart.set(df)
chart.show()

last_time = df.iloc[-1]["date"] + datetime.timedelta(days=1)
last_price = df.iloc[-1]["close"]

while True:
    time.sleep(0.1)

    change_percent = 0.002
    change = last_price * random.uniform(-change_percent, change_percent)
    new_price = last_price + change
    new_time = pd.to_datetime(last_time) + datetime.timedelta(hours=1)

    tick = pd.Series({"time": new_time, "price": new_price})

    chart.update_from_tick(tick)

    last_price = new_price
    last_time = new_time
```

随机数据基于最新的价格和时间生成，重点就是其中这一段代码：

```python
change_percent = 0.002
change = last_price * random.uniform(-change_percent, change_percent)
new_price = last_price + change
new_time = pd.to_datetime(last_time) + datetime.timedelta(hours=1)
```

将其组织成 lightweight-charts 要求的 tick 数据格式传递给 chart 即可。

```python
tick = pd.Series({"time": new_time, "price": new_price})
chart.update_from_tick(tick)
```

动图效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-05.gif)

## 指标创建

指标 Indicator 是交易图表不可缺少的部分，用 lightweight-charts 要实现指标只要通过 chart 的方法 `create_line` 创建一条 line，设置 line 数据即可。如果你的指标不是线条，而是直方图，Chart 也提供了 `create_histogram`。

我就以简单移动平均作为演示吧。指标计算函数如下：

```python
def sma(df, period: int = 50):
    return pd.DataFrame(
        {"time": df["date"], f"SMA {period}": df["close"].rolling(window=period).mean()}
    ).dropna()
```

从如上的计算函数可知，提供给 line 的数据要求包含时间和指标值，而指标值的列名要和 Line 的名称相同。主函数中新增两行代码引入指标 Line：

```python
# 保持与 sma 返回的 SMA {period} 相同
line = chart.create_line("SMA 30")
line.set(sma(df, period=30))
```

效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-06.png)

如果要实时更新的话，计算后续数值并通过 `line.update` 方法更新上去即可。

SMA 均线还是主图指标，如果希望在副图上显示指标，可通过 `chart.create_subchart` 创建副图，在其中显示副图指标。

核心代码如下所示：

```python
chart = Chart(inner_height=0.7)
chart.time_scale(visible=False) # 将主图的时间轴隐藏

# 获取数据省略

# sync 参数保持与主图同步
rsi_chart = chart.create_subchart(height=0.3, width=1, sync=True)
rsi_line = rsi_chart.create_line("RSI")
rsi_line.set(rsi(df))
```

如下 rsi 计算函数部分，其中的 RSI 指标用的是 talib 实现。

```python
def rsi(df, period: int = 14):
    return pd.DataFrame(
        {"time": df["date"], "RSI": talib.RSI(df["close"], timeperiod=14)}
    )
```

效果如下：


![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-08.png)

## 启用图例

我们已经添加了指标，但只看到线条，如想查看实际数值，通过 `chart.legend(True)` 启用图例即可。

```python
chart.legend(True) # 主图
rsi_chart.legend(True) # 副图
```

效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-07-v2.png)

## 多图表支持

前面通过 `create_subchart` 创建副图来展示副图指标，其实我们还可以基于它创建多图表。交易员可能要在不同的图表上展示不同周期或不同资产的行情，以便于寻找交易机会。

我将以在不同图表展示统一个资产的日线、周线和月线为例，演示它的使用。

```python
chart = Chart(inner_width=0.5, inner_height=0.5)
weekly_chart = chart.create_subchart(width=0.5, height=0.5, sync=True)
monthly_chart = chart.create_subchart(width=1, height=0.5, sync=True)

df = pd.read_csv("D_000001.SZ.csv")
weekly_df = pd.read_csv("W_000001.SZ.csv")
monthly_df = pd.read_csv("M_000001.SZ.csv")

chart.set(df)
weekly_chart.set(weekly_df)
monthly_chart.set(monthly_df)

chart.show(block=True)
```

如上代码创建了三个周期的 chart，分别展示日线、周线和月线。其中的周线和月线图表都是基于日线 chart 创建的。它默认是按从左到右从上到下排列的，创建的时候注意设置 chart 的宽高即可。

最终效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-05/2024-05-10-lightweight-charts-python-09.png)

我在测试时发现它的同步有点问题，好像只能在父子间同步,即在日线可以同步周线和月线，在周线或月线可以同步日线，但不能保持周线和月线的同步。或许要看 JS 库才能解决了吧?

## 其他更多

更多内容就不一一展开了，如果大家有兴趣的话，可留言说明，我可以继续完成其他部分，如周期切换、按 symbol 搜索，指标选择自定义等等，甚至与实盘框架集成，实盘下单，如 vnpy 实盘框架，vnpy 作者自研了一个图表，但功能有限，lightweight-charts 也可直接与 PySide6 集成，集成到 vnpy 框架中。

有了这个库，我们可以做一些自己想要的能力，如手动回放回测、或者自动策略与手动相结合的回测，一切尽在掌握。毕竟，自己编程，才能丰衣足食。
