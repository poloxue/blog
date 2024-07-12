---
title: "使用 Akshare 下载国内的期货（主力连续）、股票和指数的历史行情数据"
date: 2024-07-12T18:10:16+08:00
draft: false
comment: true
description: "本文介绍如何使用 akshare 下载国内的期货、股票和指数的历史行情数据。"
---

本文介绍如何使用 akshare 下载国内期货、股票和指数的历史行情数据。Akshare 是一个丰富的金融数据查询的 Python 库，通过爬虫实现了大量的金融数据接口，非常便于我们日常分析研究使用。

本文将详细介绍如何使用 Akshare 下载期货、股票和指数数据，并提供完整的代码示例，以求大家能快速拿到数据少走弯路。

本文的初衷是为了下载期货合约主要是主力连续合约的历史行情数据，用于回测分析使用，然后顺便延展到指数和股票的行情数据。

最终成果是封装一个定义如下的函数，即日线历史行情的数据下载函数。

```python
history_bar(symbol, length=100)
```

## 安装 Akshare

请确保已安装 Akshare，如下命令安装：

```bash
pip install akshare
```

了解它的更多数据，可查看它的文档：[akshare documentation](https://akshare.akfamily.xyz/platform.html)

## 下载期货数据

我使用 akshare 的重要原因就是它提供了下载期货的主力或连续合约的历史数据。开始我想 tushare 实现，映射每天的主力合约并对应获取其历史数据和拼接，这个过程本身就比较繁琐。再就是 tushare 的频率限制严重，测试几次就没有额度了。

akshare 中的主力和连续合约的数据是从新浪财经下载的，它的 symbol 名称与标准的是不一致的，不过可以通过函数 `futures_display_main_sina` 拿到映射关系。具体我不演示了，我搞了一份表格，便于我平时查看。

如需查看，可访问 [新浪期货名称关系表格](https://gist.github.com/poloxue/89395ef4001f244c4ad872ea9cfa1cb8)。

有了主力和连续合约的 symbol 名称，就可以使用 `futures_main_sina` 函数下载历史数据。

```python
data = aks.futures_main_sina(
    symbol="C0",
    start_date="20230101"
)
```

如上的 C0 表示的是玉米主力合约。

我们可以定义一个函数，参数接受 symbol 名称和长度，输出结果其转换为易读的格式。

```python
def future_history_bar(symbol, length=100):
    start_date = datetime.now() - timedelta(days=length * 2)
    data = aks.futures_main_sina(
        symbol=symbol, start_date=start_date.strftime("%Y%m%d")
    )
    data = data.rename(
        columns={
            "日期": "date",
            "开盘价": "open",
            "最高价": "high",
            "最低价": "low",
            "收盘价": "close",
            "成交量": "volume",
            "持仓量": "open_interest",
            "动态结算价": "settle_price",
        }
    )
    data["open"] = data["open"].astype(float)
    data["high"] = data["high"].astype(float)
    data["low"] = data["low"].astype(float)
    data["close"] = data["close"].astype(float)
    return data.iloc[-length:]
```

上面的参数和输出的定义看个人习惯，我比较习惯这样的使用方式，便于我计算指标。

## 下载股票数据

A 股股票数据可使用 `stock_zh_a_hist` 下载，它的输入参数包括股票代码、周期、复权方式和起始日期等参数。

```python
data = aks.stock_zh_a_hist(
    symbol="000001",
    period="daily",
    adjust="qfq",
    start_date="20230101"
)
```

我们的习惯，一般默认前复权即可。

为了使用方便，同样可定义一个函数，易于使用：

```python
def stock_history_bar(symbol, length=100):
    start_date = datetime.now() - timedelta(days=length * 2)
    data = aks.stock_zh_a_hist(
        symbol=symbol,
        period="daily",
        adjust="qfq",
        start_date=start_date.strftime("%Y%m%d"),
    )
    return data.rename(
        columns={
            "日期": "date",
            "股票代码": "symbol",
            "开盘": "open",
            "最高": "high",
            "最低": "low",
            "收盘": "close",
            "成交量": "volume",
        }
    )[
        [
            "date",
            "symbol",
            "open",
            "high",
            "low",
            "close",
            "volume",
        ]
    ].iloc[
        -length:
    ]
```

我们可以定义一个函数，下载特定指数的历史数据，并将其转换为更易读的格式：

## 下载指数数据

A 股指数数据可通过 `index_zh_a_hist` 函数实现。输入参数包括指数代码、周期和起始日期等参数。

```python
data = aks.index_zh_a_hist(
    symbol="000001",
    period="daily",
    start_date="20230101",
    end_date="20231231"
)
```

将其实现为 history_bar 函数的形式，便于我的使用。如下所示：

```python
def index_history_bar(symbol, length=100):
    start_date = (datetime.now() - timedelta(days=length * 2)).strftime("%Y%m%d")
    end_date = datetime.now().strftime("%Y%m%d")
    data = aks.index_zh_a_hist(
        symbol=symbol,
        period="daily",
        start_date=start_date,
        end_date=end_date,
    )
    return data.rename(
        columns={
            "日期": "date",
            "开盘": "open",
            "最高": "high",
            "最低": "low",
            "收盘": "close",
            "成交量": "volume",
        }
    )[
        [
            "date",
            "open",
            "high",
            "low",
            "close",
            "volume",
        ]
    ].iloc[
        -length:
    ]
```

现在，我们就已经有了三种不同交易品种的历史数据获取方法。

## 主函数

最后，还可以定义一个主函数，根据输入的类型下载相应的历史数据：

```python
def history_bar(symbol, length=100, equity_type="stock"):
    if equity_type == "stock":
        return stock_history_bar(symbol, length=length)
    elif equity_type == "future":
        return future_history_bar(symbol, length=length)
    elif equity_type == "index":
        return index_history_bar(symbol, length=length)
    else:
        raise ValueError(f"Unsupported equity type: {equity_type}")
```

现在，是不是使用起来就非常方便了。

## 示例使用

通过以下示例演示如何调用这些函数来获取不同品种的历史行情数据：

获取上证数据数据：

```python
symbol = "000001"  # 上证指数
length = 100
equity_type = "index"
data = history_bar(symbol, length, equity_type)
print(data)
```

获取平安银行数据：

```python
symbol = "000001"  # 平安银行
length = 100
equity_type = "stock"
data = history_bar(symbol, length, equity_type)
print(data)
```

获取甲醇行情数据：

```python
symbol = "MA0"  # 甲醇主连
length = 100
equity_type = "stock"
data = history_bar(symbol, length, equity_type)
print(data)
```

通过以上代码，现在就有了轻松下载股票、指数和期货历史数据的能力。这为数据分析提供了有力的支持。

`history_bars` 的代码请查看 [hist_data.py](https://gist.github.com/poloxue/a913534bc92c7fb29f0ffab5647a8ab3)。

现在有了数据，就可以实现任何其他我们像实现的事情，如计算指标发送告警，绘制图表，如果有交易 API 接口，可直接报单，不过这个前提是有了好的策略。

## 指标计算

简单演示下指标计算，如 rsi 上穿 30 可能表示短期价格有反弹，就可以发个报警，增加对这个标的的观察。

指标计算可用 talib 实现，如下所示：

```python
import talib
data = history_bar("000001", length=100, equity_type="stock")
rsi = talib.RSI(data['close'].values)

rsi0 = rsi[-1]
rsi1 = rsi[-2]

crossover = rsi0 > 30 and rsi1 < 30
if crossover:
  print("触发报警条件")
```

希望本文对您有所帮助。如果有任何问题或建议，欢迎随时联系我。

