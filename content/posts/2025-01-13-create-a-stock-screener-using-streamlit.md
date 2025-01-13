---
title: "使用 Streamlit 打造一个股票筛选分析工具"
date: 2025-01-13T16:20:35+08:00
draft: false
comment: true
description: "本文将通过 Python、Streamlit 和 Tushare，搭建一个简单易用的股票筛选器，它不仅可以筛选股票，还能查看详细数据并生成动态 K 线图，让你对股票市场有更全面的了解。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-01/2025-01-13-create-a-stock-screener-using-streamlit-00.png)

本文将介绍如何通过 Python、Streamlit 和 Tushare，搭建一个简单易用的股票筛选器，它不仅可以筛选股票，还能查看某个股票的详细数据生成动态 K 线图，让你对股票市场有更全面的了解。

## **概述**

这个股票筛选器提供将提供几个筛选能力，包括市值范围、行业、地域条件。还支持股票的 K 线图，帮助用户直观分析价格走势。

效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-01/2025-01-13-create-a-stock-screener-using-streamlit-01.gif)

这个小工具的实现离不开一个关键的 Python 开源库 - **Streamlit**。它可以快速构建交互式 Web 应用，只需几行代码，即可分享你的数据。对 streamlit 有兴趣可以深入了解下，这里不展开介绍。


如果不想看下面的详细步骤可直接从 [github gist](https://gist.github.com/poloxue/b45324e478d97110e8f9969225e33e1a) 拿到代码，完成第一部分的准备即可运行。

想进一步扩展的话，还可以添加技术指标（如均线、RSI）、多只股票价格走势对比、实时数据更新、模拟投资组合功能甚至是按筛选分析结果生成股票分析报告等功能。

---

接下来，快速逐步讲解代码的实现过程，尽量每一步都清晰易懂。

## 准备开始

安装要用到 python 库：

```bash
pip install tushare ploty streamlit
```

并提前导入依赖包：

```python
import streamlit as st
import tushare as ts
import pandas as pd
import plotly.graph_objects as go
```

通过 `pro = ts.pro\_api()` 配置了 Tushare 接口，为数据获取做好准备。

```python
ts.set_token("Your API Token")
pro = ts.pro_api()
```

注：tushare pro 依赖于 API Key，如果没有的话，到 tushare pro 平台注册获取。

---

## **股票数据**

为了完成股票数据的筛选，要用到 tushare 的两个，分别是 `stock_basic` 股票基础数据和 `stock_daily` 每日最新指标，这里有要用到的市值数据。

### 基础信息

首先，获取所有上市股票的基本信息，包括股票代码、名称、行业、地域和上市日期。如果某些行业或地域数据缺失，用 "Unknown" 填补空值。

```python
@st.cache_data
def get_stock_list():
    df = pro.stock_basic(
        exchange="",
        list_status="L",
        fields="ts_code,symbol,name,area,industry,list_date",
    )
    df["area"] = df["area"].fillna("Unknown")
    df["industry"] = df["industry"].fillna("Unknown")
```

为了提高性能和减少重复计算，这里用了 `@st.cache_data`  装饰器缓存数据结果，股票的基础数据是不怎么变化的，这样可避免每次交互重复加载数据。

### **市值数据**

为了筛选高市值股票，调用每日基础数据接口，获取市值数据，并将其与基本信息合并。

```python
market_cap_data = pro.daily_basic()
market_cap_data["total_mv"] = market_cap_data["total_mv"] / 10000  # 转换为"亿"
df = pd.merge(df, market_cap_data, on="ts_code", how="left")
df = df.rename(columns={"total_mv": "market_cap"})
```

到这里，我们生成了一份包含公司基本信息和市值的完整股票列表，为后续筛选做好准备。

---

## **界面和筛选条件**

接下来，创建用户界面和筛选项。

```
st.title("Stock Screener")
st.sidebar.header("Filter Options")
```

侧边栏设置以下筛选条件：

- **市值范围**：设置筛选股票的最小和最大市值。
- **行业和地域**：按行业或地域选择股票。
- **上市日期范围**：筛选符合时间条件的公司。

### 筛选实现

以下代码实现了这些过滤功能：

```python
# 股票的基本信息
stocks = get_stock_list()

# 筛选市值范围
min_market_cap = st.sidebar.number_input("Minimum Market Cap (亿)", min_value=0, value=100)
max_market_cap = st.sidebar.number_input("Maximum Market Cap (亿)", min_value=0, value=1000)

# 筛选行业和地域
industry_list = stocks["industry"].unique().tolist()
selected_industry = st.sidebar.multiselect("Select Industry", industry_list)

area_list = stocks["area"].unique().tolist()
selected_area = st.sidebar.multiselect("Select Area", area_list)
```

根据这些条件，我们过滤股票数据：

```python
filtered_stocks = stocks.copy()
filtered_stocks = filtered_stocks[
    (filtered_stocks["market_cap"] >= min_market_cap) &
    (filtered_stocks["market_cap"] <= max_market_cap)
]
if selected_industry:
    filtered_stocks = filtered_stocks[filtered_stocks["industry"].isin(selected_industry)]
if selected_area:
    filtered_stocks = filtered_stocks[filtered_stocks["area"].isin(selected_area)]
```

这样可以看到符合条件的股票列表。

---

### **结果显示**

我们继续通过 `streamlit` 的表格形式展示筛选后的股票列表，按照市值大小排序。

```python
display_df = filtered_stocks.copy()
display_df["market_cap"] = display_df["market_cap"].round(2)
display_df = display_df.sort_values(by="market_cap", ascending=False)
st.dataframe(display_df)
```

这样就可以清晰地看到结果，并快速找到目标股票。

---

## **单只股票详情**

有了筛选的股票列表后，还可以继续扩展，查看单只股票的情况，如蜡烛图、价格表格。

### **日线数据**

先通过 Tushare 接口获取用户选择的股票的日线数据，并根据日期范围过滤。

```python
@st.cache_data
def get_daily_data(ts_code):
    return pro.daily(ts_code=ts_code)

daily_data = get_daily_data(stock_code)
daily_data["trade_date"] = pd.to_datetime(daily_data["trade_date"])
daily_data = daily_data.sort_values("trade_date", ascending=True)
```

### 筛选界面

配置筛选条件，包括股票选择框和日期范围。

```python
# Show details for selected stock
selected_stock = st.selectbox("Select a stock to view details", filtered_stocks["name"])

today = pd.Timestamp.today()
col1, col2 = st.columns(2)
with col1:
    start_date = st.date_input(
        "Start date", value=today - pd.Timedelta(days=365), max_value=today
    )
with col2:
    end_date = st.date_input(
        "End date", value=today, min_value=start_date, max_value=today
    )

```
### 筛选逻辑

如果设置了 `selected_stock` 进入到筛选逻辑中。

```python
if selected_stock:
    stock_code = filtered_stocks[filtered_stocks["name"] == selected_stock][
        "ts_code"
    ].values[0]
    daily_data = get_daily_data(stock_code)

    daily_data["trade_date"] = pd.to_datetime(daily_data["trade_date"])
    daily_data = daily_data.sort_values("trade_date", ascending=True)

    # 按时间过滤
    daily_data = daily_data[
        (daily_data["trade_date"] >= pd.Timestamp(start_date))
        & (daily_data["trade_date"] <= pd.Timestamp(end_date))
    ]

    # 检查缺失数据
    if daily_data[["open", "high", "low", "close"]].isnull().values.any():
        st.warning("Some price data is missing - filling with previous values")
        daily_data[["open", "high", "low", "close"]] = daily_data[
            ["open", "high", "low", "close"]
        ].ffill()
```

### **绘制 K 线图**

使用 Plotly 绘制动态 K 线图，让用户更直观地观察股票的价格变化。

```python
fig = go.Figure(
    data=[
        go.Candlestick(
            x=daily_data["trade_date"],
            open=daily_data["open"],
            high=daily_data["high"],
            low=daily_data["low"],
            close=daily_data["close"],
            increasing_line_color="green",
            decreasing_line_color="red",
        )
    ]
)
fig.update_layout(title=f"{selected_stock} Candlestick Chart", xaxis_title="Date", yaxis_title="Price")
st.plotly_chart(fig, use_container_width=True)
```

---

### **总结与扩展**

通过以上步骤简单构建了一个股票筛选分析工具，它支持按市值、行业、地域和上市日期筛选股票，可视化 K 线图。


希望本文对你有用，开始构建属于你的股票分析工具吧！


