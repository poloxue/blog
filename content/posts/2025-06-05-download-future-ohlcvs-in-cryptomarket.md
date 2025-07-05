---
title: "加密货币交割合约的全量行情数据下载"
date: 2025-06-05T12:00:00+08:00
draft: true
comment: true
description: "本文介绍如何从数字货币交易所下载全量的交割合约行情数据。"
---

本文介绍如何从数字货币交易所下载全量的交割合约行情数据。

对现货和永续合约而言，很轻松就能找到完整的历史 K 线。但当涉及交割合约，尤其是已下线的合约，就会遇到障碍。

- 如何获得历史上交割合约的信息？
- 如何获取这些历史合约的行情数据？

这篇文章将探索下载交割合约行情数据的方案，尽可能通过交易所官方渠道，如有不足，想办法通过其他数据聚合平台弥补不足。

---

## 为什么常规方式无法获取完整数据？

大多数交易所（如 Binance、OKX、Bybit 等）在 API 中仅保留当前正在交易的合约。即使你使用 `ccxt` 获取 futures 合约列表，只能拿到当前在交易的合约。

```python
import ccxt

exchange = ccxt.okx()
markets = exchange.load_markets()
futures = [m for m in markets.values() if market['future']]
```

这段代码返回的合约都是**当前仍在交易**的合约。

```python
for m in futures:
    print(m["id"])
```

输出：

```bash
BTC-USD-250606
BTC-USD-250613
...
BTC-USDC-250627
ETH-USDC-250627
```

对于已经下线的合约（例如 BTC-USD-220930），API 不再返回其基本信息，自然也无法通过 ccxt 获取其历史 K 线。

---

接下来，我将尝试实现下载 OKX 和 Binance 的交割合约数据。

## OKX 的交割合约数据下载 

首先，获取历史交割合约的基本信息。

我搜了半天，也没有在 OKX 的官网上找到交割合约历史数据的下载方法。一番搜索后，在一个名为 tardis.dev 的三方平台找到了这个数据，可免费获取。

```python
import json
import requests
```

OKX 提供了交割合约的 [历史数据下载页面](https://www.okx.com/trade-markets/downloads)，包括：

* 已下线合约的 K 线（CSV 文件）
* 上线与下线时间的命名信息可从文件名中提取

你可以通过脚本批量下载这些文件：

```bash
wget https://static.okx.com/cdn/okex/traderecords/futures/Delivery-BTC-USD-20220930.csv
```

再配合合约名中的时间提取上线/交割日期。

#### Binance 的历史合约归档页面

Binance 提供 [数据归档页面](https://data.binance.vision/?prefix=data/delivery/)，可按日期、合约品种下载：

* 支持下载 `klines`, `trades`, `aggTrades` 等数据
* 命名格式中包含合约交割时间，如 `BTCUSD_220930`

示例：

```bash
wget https://data.binance.vision/data/delivery/klines/BTCUSD_220930/1d/BTCUSD_220930-1d-2022-06-01.zip
```

然后可用 pandas 解压和处理：

```python
import pandas as pd
import zipfile

with zipfile.ZipFile("BTCUSD_220930-1d-2022-06-01.zip", 'r') as zip_ref:
    zip_ref.extractall(".")

df = pd.read_csv("BTCUSD_220930-1d-2022-06-01.csv", header=None)
```

---

### 2. 利用 `ccxt` 获取补充数据

对于当前活跃合约（如 BTC-USD-240628），仍然可以使用 `ccxt` 来获取历史行情：

```python
ohlcv = exchange.fetch_ohlcv('BTC-USD-240628', timeframe='1d', since=timestamp)
```

因此完整策略是：

* 对于已下线的合约：使用交易所的历史归档页面（Binance、OKX）
* 对于当前活跃合约：使用 `ccxt.fetch_ohlcv()`

---

## 三、如何批量构建历史合约的列表

举例：以 Binance 为例，你可以从归档路径中获取所有历史交割合约名称

```python
import requests
from bs4 import BeautifulSoup

def list_binance_delivery_contracts():
    url = "https://data.binance.vision/data/delivery/klines/"
    res = requests.get(url)
    soup = BeautifulSoup(res.text, "html.parser")
    return [a['href'].strip("/") for a in soup.find_all("a") if "BTCUSD_" in a['href']]

contracts = list_binance_delivery_contracts()
print(contracts)
```

---

## 四、将数据整合到回测框架（如 Backtrader）

一旦你拿到格式标准的历史 K 线数据（CSV），就可以喂给如 `backtrader`、`zipline`、`bt` 等框架：

```python
import backtrader as bt

class MyStrategy(bt.Strategy):
    def next(self):
        print(self.data.close[0])

data = bt.feeds.GenericCSVData(
    dataname='BTCUSD_220930-1d.csv',
    timeframe=bt.TimeFrame.Days,
    dtformat='%Y-%m-%d',
    datetime=0,
    open=1, high=2, low=3, close=4, volume=5,
)

cerebro = bt.Cerebro()
cerebro.adddata(data)
cerebro.addstrategy(MyStrategy)
cerebro.run()
```

你可以按年份合并多个合约形成“连续合约数据”，模拟合约换月（注意需要前复权逻辑）。

---

## 五、结语

数字货币的交割合约市场虽然不像永续合约那样活跃。`ccxt` 本身虽然无法提供已下线合约的全量数据，但结合交易所公开数据、网页解析与本地存储构建体系，依然可以实现完整的交割合约历史数据系统。后续你也可以考虑构建自动化数据同步与数据湖，作为策略回测平台的数据源。

