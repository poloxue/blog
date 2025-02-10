---
title: "如何通过 Python 获取加密货币的历史实时行情"
date: 2025-02-10T15:08:29+08:00
draft: false
comment: true
description: "如果你想参与加密货币市场，获取实时、准确的市场数据就是重要的第一步。"
---

如果你想参与加密货币市场，获取实时、准确的市场数据就是重要的第一步。

不同于传统金融市场，加密货币市场上的交易所有两类，分别 DEX 和 CEX，即中心化交易所和去中心化交易所。两者的区别这里就不展开了，我们大部分人接触的一般都是中心化交易所。和传统金融市场一样，加密货币市场上的数据，我们也可找专业的数据提供商，但基本都是付费的。

本文将介绍如何通过 Python 从中心化交易所拿到行情数据，包括历史和实时数据，包括直接从 websocket 监控数据，可用来实时监听不对交易对间的价差数据。

## 概要说明

从去中心化交易所拿到交易所数据，可直接通过访问它的 HTTP 和 Websocket 接口，如欧易 OKX 的接口，查看它的官方文档：[欧易 API 文档](https://www.okx.com/docs-v5/zh/)，其他每个交易所也都有这类文档，可查看其支持的 API 接口。

不过市面上的中心化交易所很多，而且如果要获取账户级别的信息，还有经过认证流程。本文为了简化流程，覆盖更多的去中心化交易所，我会使用一个非常流行的 Python 库 ccxt 演示，从而把重点集中在数据的获取方法上。

## 什么是 ccxt

ccxt，全称 CryptoCurrency eXchange Trading，这个开源库能让你用统一的代码访问全球上百家数字货币交易所。它支持多个编程语言，如 Javascript、Python 和 PHP。

ccxt，除了可用来获取数据，还可以支持直接下单、订单管理、访问仓位、账户等，用来实现一个自动化交易人。

---

接下来开始正文吧！

本文将从最简单的价格获取到复杂的订单簿处理，带你一步步深入。


## 安装 ccxt

打开终端，输入如下命令，安装 ccxt 库：

```bash
pip install ccxt
```

## 初始化交易所

以币安为例，先创建一个交易所对象：

```python
import ccxt

exchange = ccxt.binance({
    'enableRateLimit': True,  # 必须开启！防止被交易所封IP
    'timeout': 15000          # 超时设为15秒
})
```

交易所的API都有严格的调用频率限制，`enableRateLimit` 会启用频率限制能力，防止触发交易所的 API 调用频率。

如果遇到网络问题，也可通过设置 `proxies` 切换网络环境：

```python
exchange = ccxt.okx({
    "proxies": {
        "http": "http://127.0.0.1:7890",
        "https": "http://127.0.0.1:7890",
    }
})
```

获取交易所上的所有品种：

```python
markets = exchange.load_markets()
print(list(markets.keys)[:10])
```

输出如下：

```bash
['ETH/BTC', 'LTC/BTC', 'BNB/BTC', 'NEO/BTC', 'QTUM/ETH', 'EOS/ETH', 'SNT/ETH', 'BNT/ETH', 'BCC/BTC', 'GAS/BTC']
```

在开始其他操作前，这一步是必要的，否则你可能会遇到品种不存在的问题。

---

## 实时价格

首先，示例如何通过 ccxt 获取某个品种当前的价格，主要依赖于 `fetch_ticker` 函数接口。

示例代码，读取 `BTC/USDT` 交易对的最新价。

```python
ticker = exchange.fetch_ticker('BTC/USDT')

print(f"""
实时价格：
- 最新价：{ticker['last']} USDT
- 24小时最高：{ticker['high']}
- 24小时成交量：{ticker['quoteVolume']} USDT
""")
```

输出：

```
实时价格：
- 最新价：97004.87 USDT
- 24小时最高：97640.02
- 24小时成交量：1743212485.2952676 USDT
```

ticker 的结构体可查看官方文档 [Ticker Structure](https://docs.ccxt.com/#/README?id=ticker-structure)。

```bash
{{
    'symbol':        string 类型，表示市场的交易对（例如：'BTC/USD', 'ETH/BTC' 等），
    'info':          原始未修改的交易所 API 回复信息（未经解析的原始数据），
    'timestamp':     integer 类型，64 位 Unix 时间戳，单位为毫秒，自 1970 年 1 月 1 日起，
    'datetime':      ISO8601 格式的日期时间字符串（带毫秒），
    'high':          float 类型，市场的最高价格，
    'low':           float 类型，市场的最低价格，
    'bid':           float 类型，当前最佳买入价，
    'bidVolume':     float 类型，当前最佳买入量（可能缺失或未定义），
    'ask':           float 类型，当前最佳卖出价，
    'askVolume':     float 类型，当前最佳卖出量（可能缺失或未定义），
    'vwap':          float 类型，加权平均价格，
    'open':          float 类型，开盘价，
    'close':         float 类型，最后交易价格（当前周期的收盘价），
    'last':          float 类型，等同于 `close`，为方便重复表示，
    'previousClose': float 类型，上一个周期的收盘价，
    'change':        float 类型，绝对变化，`last - open`，
    'percentage':    float 类型，相对变化，`(change/open) * 100`，
    'average':       float 类型，平均价格，`(last + open) / 2`，
    'baseVolume':    float 类型，过去 24 小时内交易的基础货币的交易量，
    'quoteVolume':   float 类型，过去 24 小时内交易的报价货币的交易量，
}
```

简言之，`fetch_ticker` 返回的是某个交易对的当前最新的市场数据。

---

## 历史行情

除了最新行情，历史行情对于交易也非常有帮助，如我们用它来计算指标，识别模式，产生交易信号。

如下是获取 `BTC/USDT` 的日线数据的代码：

```python
# 获取最近3天的日线数据
ohlcvs = exchange.fetch_ohlcv(
    symbol='BTC/USDT',
    timeframe='1d',  # 可选：1m, 5m, 1h, 1d 等
    limit=5          # 获取条数
)
print(ohlcvs)
```

参数设置获取 `BTC/USDT` 的历史行情，时间周期为 `1d`，数量为 5。

输出：

```bash
[[1739138760000, 94897.63, 95052.0, 94884.44, 95047.09, 17.96677],
 [1739138820000, 95047.1, 95128.75, 95047.1, 95118.36, 12.27411], 
 [1739138880000, 95118.36, 95135.99, 95051.65, 95122.51, 10.40116],
 [1739138940000, 95122.52, 95178.69, 95104.42, 95178.57, 7.81859],
 [1739139000000, 95178.57, 95246.13, 95160.0, 95246.13, 48.00226]]
```

列表中的每个元素依然是一个列表，从左到右分别是时间戳、开盘价、最高价、最低价、收盘价和交易量。

可以将其转为更加易读的 pandas 格式，如下所示；

```python
data = pd.DataFrame(ohlcvs, columns=["timestamp_ms", "open", "high", "low", "close", "volume"])
print(data)
```

输出：

```bash
    timestamp_ms      open      high       low     close    volume
0  1739139000000  95178.57  95246.13  95160.00  95246.13  48.00226
1  1739139060000  95246.12  95436.12  95230.02  95429.63  24.59766
2  1739139120000  95429.64  95429.64  95282.44  95371.00  44.81550
3  1739139180000  95371.00  95443.28  95371.00  95433.12  12.77712
4  1739139240000  95433.12  95527.79  95433.12  95436.70  15.98278
```

如果想设计一个高频策略，可能还要拿到分钟级别数据，很多交易所都支持拿到分钟级别数据，甚至是全量的分钟数据。

假设，获取 `BTC/USDT` 的 1 分钟数据，只需修改 `fetch_ohlcv` 的 `timeframe` 参数即可。

示例代码:

```python
# 获取最近20根15分钟K线
ohlcvs = exchange.fetch_ohlcv('BTC/USDT', '1m', limit=20)
```

## 订单簿（市场深度）

除了行情数据，如果你拿到订单薄数据，ccxt 也可以做到，只要调用 `fetch_order_book` 即可。

一个简单示例：

```python
order_book = exchange.fetch_order_book('BTC/USDT', limit=100)

best_bid = order_book['bids'][0][0]  # 买一价
best_ask = order_book['asks'][0][0]  # 卖一价
print(f"买一价：{best_bid}, 卖一价：{best_ask}, 价差：{best_ask - best_bid}")
```

输出：

```
买一价：97141.35, 卖一价：97141.36, 价差：0.00999999999476131
```

其中，`fetch_order_book` 可用于指定获取深度，像 `binance` 的接口默认是 100，而最多可以获取到 5000 的订单薄深度。

## 实时监听

前面介绍完的几种数据接口，都是我们主动发起请求，对于一些实时性要求非常高的场景，轮询延迟大，丢掉一些交易机会，且可能触发交易所的频率限制。ccxt 还提供了一个 pro 版本 `ccxt.pro`，可直接通过 WebSocket 实时监听。

前面的接口，我们简单修改下，就可以实现实时监听的效果。

```python
import asyncio
import ccxt.pro as ccxtpro

exchange = ccxtpro.binance({
    'enableRateLimit': True,  # 必须开启！防止被交易所封IP
    'timeout': 15000,         # 超时设为15秒
})

async def watch_order_book():
    while True:
        order_book = await exchange.watch_order_book('BTC/USDT')
        best_ask = order_book["asks"][0][0]
        best_bid = order_book["bids"][0][0]
        print(f"order_book, 价差:{best_ask-best_bid}")

async def watch_ticker():
    while True:
        ticker = await exchange.watch_ticker('BTC/USDT')
        print(f"ticker, 最新价: {ticker['last']}")

async def main():
    await asyncio.gather(watch_order_book(), watch_ticker())

if __name__ == "__main__":
    asyncio.run(main())
```

输出：

```bash
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
order_book, 价差:0.00999999999476131
ticker, 最新价: 97228.01
...
```

现在就能实时监控，不断输出新的数据。

---

## 总结

现在，就已经掌握了如何利用 ccxt 获取市场行情，接下来可以：  

2. **监控价差**：实时计算不同交易所之间的套利机会
3. **对接交易**：用ccxt的 `create_order` 方法实现自动交易  

最后，希望本文对你有所帮助，感谢阅读。