---
title: "使用 Akshare 下载国内的期货（主力连续）、股票和指数的历史行情数据"
date: 2024-07-12T18:10:16+08:00
draft: false
comment: true
description: "本文介绍如何使用 akshare 下载国内的期货、股票和指数的历史行情数据。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-07/2024-07-12-download-quote-data-in-china-market-00.webp)

本文介绍如何使用 akshare 封装一个易用的下载国内期货、期货指数、股票和股票指数的历史行情数据接口。

目标接口的定义如下所示：

```python
history_bars(symbol, length=100, equity_type="future")
```

这么封装也是为了便于使用。

如果不了解 akshare，我已经写过一篇关于 akshare 下载金融数据的文章：[通过 python 获取金融数据-akshare](https://www.poloxue.com/posts/2024-11-02-financial-market-data/)。

## 安装 Akshare

请确保已安装 Akshare，如下命令安装：

```bash
pip install akshare
```

了解它的更多数据，可查看它的文档：[akshare documentation](https://akshare.akfamily.xyz/platform.html)

## 下载期货数据

akshare 中的主力和连续合约的数据是从新浪财经下载的，要请求行情接口，首先要知道品种对应的 symbol 名称。它的 symbol 名称可以通过函数 `futures_display_main_sina` 拿到映射关系。

```python
import akshare as aks
print(aks.futures_display_main_sina())
```

输出：

```bash
   symbol exchange         name
0      V0      dce        PVC连续
1      P0      dce        棕榈油连续
2      B0      dce         豆二连续
3      M0      dce         豆粕连续
4      I0      dce        铁矿石连续
..    ...      ...          ...
72    IC0    cffex  中证500指数期货连续
73    TS0    cffex    2年期国债期货连续
74    IM0    cffex   中证连续指数期货连续
75    SI0     gfex        工业硅连续
76    LC0     gfex        碳酸锂连续

[77 rows x 3 columns]
```


现在将 symbol 传递给 `futures_main_sina` 函数即可下载合约的历史数据。

如上的 C0 表示的是玉米主力合约，下载它的日线行情：

```python
data = aks.futures_main_sina(
    symbol="C0",
    start_date="20230101"
)
```

为便于统一封装到 `history_bars`，我们定义一个函数 `future_history_bars`，参数是 symbol 名称和长度，将输出结果转换为易读的格式。

```python
def future_history_bars(symbol, length=100):
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

> **提醒：**
>
> 上述的按 length 获取数据的方式没有按交易日历来，只是粗略地从当前时间往前推 `2*length` 个周期得到开始时间 `start_date`，如果 `length` 太小，例如两个周期，如果赶上假期，就可能获取不到数据，这点要注意的。
> 
> 如果想长度精确，最简单的方式就是把数据存入数据库，通过 SOL 加载。或者是拿到交易日历，按日历计算交易天数。

## 南华期货指数

我最初封装这个接口是为了回测策略，但发现期货主连数据是未复权的数据，没有考虑换仓，不符合真实交易场景。

有没有满足需求的数据？

当然有！市面上有南华期货指数，它考虑了换仓的场景，经过复权编制出的数据，符合真实交易场景，可用来回测。

akshare 的文档中提供了南华指数的接口，我测试了下。很遗憾，几个接口都不能用了。

```python
# aks.futures_index_symbol_table_nh() 南华 symbol 表格
# aks.futures_price_index_nh() 南华价格指数
# aks.futures_return_index_nh() 南华收益指数
```

我本希望这篇文章中都是免费数据，但确实没有找到好的渠道。如果确有需要，可从 tushare 下载，访问 [南华期货指数日线行情](https://www.tushare.pro/document/2?doc_id=155)。

除了使用南华期货指数，也可自己计算，前提是知道主连合约的换仓时间。在 akshare 上没发现这个数据，如需要，也可从 tushare 下载，访问[期货主力与连续合约](https://www.tushare.pro/document/2?doc_id=189)，其中包含了主连合约每天映射的实际合约。

将 tushare 的南华指数行情封装到成 `history_bars` 的形式。

```python
def nanhua_future_history_bars(symbol, length=100):
    start_date = (datetime.now() - timedelta(days=length * 2)).strftime("%Y%m%d")
    end_date = datetime.now().strftime("%Y%m%d")

    pro = ts.pro_api()
    data = pro.index_daily(ts_code=symbol, start_date=start_date, end_date=end_date)
    if not len(data):
        return

    data.dropna(inplace=True)
    data.rename(columns={"trade_date": "date", "vol": "volume"}, inplace=True)
    data.sort_values(by="date")
    data["date"] = pd.to_datetime(data["date"])
    data.set_index("date", inplace=True)

    return data
```
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
def stock_history_bars(symbol, length=100):
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
def index_history_bars(symbol, length=100):
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
def history_bars(symbol, length=100, equity_type="stock"):
    if equity_type == "stock":
        return stock_history_bar(symbol, length=length)
    elif equity_type == "future":
        return future_history_bar(symbol, length=length)
    elif equity_type == "nan_future":
        return nanhua_future_history_bar(symbol, length=length)
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
data = history_bars(symbol, length, equity_type)
print(data)
```

获取平安银行数据：

```python
symbol = "000001"  # 平安银行
length = 100
equity_type = "stock"
data = history_bars(symbol, length, equity_type)
print(data)
```

获取甲醇行情数据：

```python
symbol = "MA0"  # 甲醇主连
length = 100
equity_type = "stock"
data = history_bars(symbol, length, equity_type)
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
data = history_bars("000001", length=100, equity_type="stock")
rsi = talib.RSI(data['close'].values)

rsi0 = rsi[-1]
rsi1 = rsi[-2]

crossover = rsi0 > 30 and rsi1 < 30
if crossover:
  print("触发报警条件")
```

希望本文对您有所帮助。如果有任何问题或建议，欢迎随时联系我。

