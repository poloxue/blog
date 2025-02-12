---
title: "2025 02 12 Download Cryptocurrency Minute Candlestick"
date: 2025-02-12T12:01:03+08:00
draft: false
comment: true
description: "今天，分享一个简短的脚本，从 binance 下载全量的行情数据，包括分钟级别的 OHLCV 行情。"
---

前面写了一篇关于 [python 获取加密货币行情](https://www.poloxue.com/posts/2025-02-10-crypto-currency-market-data-using-ccxt/) 的数据文章，但如果要获取数据，还要亲自写代码，用起来不方便。

今天，分享一个简短的脚本，从 binance 下载全量的行情数据，包括分钟级别的 OHLCV 行情。

如下载 2024-01-01 到 2025-01-01 的全量分钟数据，指定如下命令即可。

```bash
python data.py --symbol BTC/USDT --start 2021-01-01 --end 2023-01-01 --timeframe 1d --save-dir ./data
```

它会将下载的数据保存为 CSV 文件，适合自己平时分析研究的时候使用。

下面把代码和过程分享一下。

---

首先，需要安装以下 Python 库：

```bash
pip install ccxt pandas click python-dateutil pytz
```

其中：

- **ccxt** 用于连接 Binance，获取行情数据；  
- **pandas** 负责数据处理和保存成 CSV 文件；  
- **click** 让代码可以用命令行的方式调用，参数传递更方便。

特别提醒：如果遇到网络问题，要自己想办法解决了。

---

如下是完整的代码，可保存为 data.py 文件。

```python
import os
import datetime
import pytz
import pandas as pd
import ccxt
import click

from dateutil.relativedelta import relativedelta
from pathlib import Path

exchange = ccxt.binance(
    {
        "enableRateLimit": True,  # 必须开启！防止被交易所封IP
        "timeout": 15000          # 超时设为15秒
        "proxies": {
            "http": os.getenv("http_proxy"),
            "https": os.getenv("https_proxy"),
        }
    }
)

def download(symbol: str, start=None, end=None, timeframe="1d", save_dir="."):
    if end is None:
        end = datetime.datetime.now(pytz.UTC)
    if start is None:
        start = end - relativedelta(years=3)

    max_limit = 1000
    since = start.timestamp()
    end_time = int(end.timestamp() * 1e3)

    # Create save directory if it doesn't exist
    Path(save_dir).mkdir(parents=True, exist_ok=True)
    
    absolute_path = os.path.join(
        save_dir, f"{symbol.replace('/', '-')}_{timeframe}.csv"
    )

    ohlcvs = []
    while True:
        new_ohlcvs = exchange.fetch_ohlcv(
            symbol, since=int(since * 1e3), timeframe=timeframe, limit=max_limit, params={"endTime": end_time }
        )
        if len(new_ohlcvs) == 0:
            break
        ohlcvs += new_ohlcvs

        since = ohlcvs[-1][0]/1e3 + 1
        print(f"下载进度：{datetime.datetime.fromtimestamp(ohlcvs[-1][0]/1e3)}\r", end="")
    print()

    data = pd.DataFrame(
        ohlcvs, columns=["timestamp_ms", "open", "high", "low", "close", "volume"]
    )
    data.drop_duplicates(inplace=True)
    data.set_index(
        pd.DatetimeIndex(pd.to_datetime(data["timestamp_ms"], unit="ms", utc=True)),
        inplace=True,
    )
    data.index.name = "datetime"
    del data["timestamp_ms"]
    data.to_csv(absolute_path)
    print(f"Data saved to: {absolute_path}")

@click.command()
@click.option("--symbol", required=True, help="Trading symbol (e.g. BTC/USDT)")
@click.option("--start", type=click.DateTime(), help="Start date (YYYY-MM-DD)")
@click.option("--end", type=click.DateTime(), help="End date (YYYY-MM-DD)")
@click.option("--timeframe", default="1d", help="Timeframe for OHLCV data (1m, 5m, 15m, 1h, 1d, etc.)")
@click.option("--save-dir", default=".", help="Directory to save the CSV file")
def main(symbol, start, end, timeframe, save_dir):
    """Download OHLCV data from Binance"""
    download(symbol=symbol, start=start, end=end, timeframe=timeframe, save_dir=save_dir)

if __name__ == "__main__":
    main()
```

---

## 功能介绍  

这个脚本通过 Binance 的 API 获取指定交易对的历史数据。默认下载最近 3 年的数据，当然你也可以通过命令行参数指定时间范围。  

时间周期可以选择 1 分钟到 1 天等多种类型，比如：  
- `1m`（1 分钟）  
- `5m`（5 分钟）  
- `1h`（1 小时）  
- `1d`（1 天）  
- `1w`（1 周）等

下载的数据会保存成 CSV 文件，文件名形如 `BTC-USDT_1d.csv`，方便后续分析。  

假设你想下载 `BTC/USDT` 交易对在 2021 年到 2023 年期间的日线数据，可以在命令行中运行：  

```bash
python data.py --symbol BTC/USDT --start 2021-01-01 --end 2023-01-01 --timeframe 1d --save-dir ./data
```

脚本会自动下载数据，并保存到 `./data` 目录下的 `BTC-USDT_1d.csv` 文件中。  

---

这个脚本算是一个小工具，虽然代码不多，但挺实用的，能省去很多重复工作。如果有需要，也可以扩展成自动化脚本，定时更新最新的数据。  
