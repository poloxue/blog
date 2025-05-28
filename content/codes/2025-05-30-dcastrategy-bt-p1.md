---
title: "定期定额回测代码"
date: 2025-05-28T17:56:10+08:00
draft: false
comment: true
description: "本文记录数字货币的定期定额的代码，基于 backtrader 实现。"
---

本文记录数字货币的定期定额的代码，基于 backtrader 实现。

完整代码如下：

```python
import os
import ccxt
import pytz
import pandas as pd
import numpy as np
import datetime

from dateutil.parser import parse as datetime_parse
from dateutil.relativedelta import relativedelta

import click
import backtrader as bt


params = {
    "enableRateLimit": True,
    "proxies": {
        "http": os.getenv("http_proxy"),
        "https": os.getenv("http_proxy"),
    },
}

exchanges = {}


def create_exchange(name):
    if name in exchanges:
        return exchanges[name]
    else:
        if not hasattr(ccxt, name):
            raise ValueError(f"Not supported exchange: {name}")
        exchanges[name] = getattr(ccxt, name)(params)
        return exchanges[name]


def symbols(market="swap.linear", quote_ccy="USDT", exchange_name="binance"):
    exchange = create_exchange(exchange_name)
    markets = exchange.load_markets()

    market_type, market_subtype = market.split(".")
    return [
        m["symbol"]
        for m in markets.values()
        if m["active"]
        and m[market_type]
        and m[market_subtype]
        and m["quoteId"] == quote_ccy
    ]


def validate_date_range(start_date, end_date):
    if end_date is None:
        end_date = datetime.datetime.now(pytz.UTC)
    elif isinstance(end_date, str):
        end_date = datetime_parse(end_date)
        end_date = end_date.replace(tzinfo=pytz.UTC)
    else:
        end_date = end_date.replace(tzinfo=pytz.UTC)

    current_date = datetime.datetime.now(pytz.UTC)
    if end_date > current_date:
        end_date = current_date

    if start_date is None:
        start_date = end_date - relativedelta(years=3)
    elif isinstance(start_date, str):
        start_date = datetime_parse(start_date)
        start_date = start_date.replace(tzinfo=pytz.UTC)
    else:
        start_date = start_date.replace(tzinfo=pytz.UTC)

    return start_date, end_date


def download(
    symbol: str, start_date=None, end_date=None, interval="1d", exchange_name="binance"
):
    start_date, end_date = validate_date_range(start_date, end_date)

    exchange = create_exchange(exchange_name)

    max_limit = 100
    since = int(start_date.timestamp() * 1e3)
    end_time = int(end_date.timestamp() * 1e3)

    ohlcvs = []
    while True:
        new_ohlcvs = exchange.fetch_ohlcv(
            symbol=symbol,
            since=since,
            timeframe=interval,
            limit=max_limit,
        )
        new_ohlcvs = [ohlcv for ohlcv in new_ohlcvs if ohlcv[0] <= end_time]
        if len(new_ohlcvs) == 0:
            break
        ohlcvs += new_ohlcvs

        since = ohlcvs[-1][0] + 1000

    columns = ["timestamp", "open", "high", "low", "close", "volume"]
    data = pd.DataFrame(ohlcvs, columns=np.array(columns))
    data.drop_duplicates(inplace=True)
    data["datetime"] = pd.to_datetime(data["timestamp"], unit="ms", utc=True)
    data.set_index("datetime", inplace=True)
    data.drop(columns=["timestamp"], inplace=True)
    data["symbol"] = symbol

    return data


def strategy_title(interval, dayoffset):
    cn_dayoffsets = ["一", "二", "三", "四", "五", "六", "日"]
    if interval == "1m":
        return f"每月({dayoffset}号)"
    elif interval == "1w":
        return f"每周(星期{cn_dayoffsets[dayoffset - 1]})"
    elif interval == "2w":
        return f"每双周(星期{cn_dayoffsets[dayoffset - 1]})"
    elif interval == "1d":
        return "每日"


def next_month_day(dt, day):
    if dt.month == 12:
        year = dt.year + 1
        month = 1
    else:
        year = dt.year
        month = dt.month + 1
    return datetime.date(year=year, month=month, day=day)


def investment_times(start_date, end_date, interval="1d"):
    days = (end_date - start_date).days
    if interval == "1d":
        return days
    elif interval == "1w":
        return int(days // 7)
    elif interval == "2w":
        return int(days // 14)
    elif interval == "1m":
        return int(days // 30)
    else:
        return 0


def start_investment_date(current_date, dayoffset, interval="1d"):
    if interval == "1d":
        return current_date
    elif interval == "1w":
        if current_date.weekday() <= dayoffset - 1:
            days_ahead = dayoffset - 1 - current_date.weekday()
            return current_date + datetime.timedelta(days=days_ahead)
        else:
            days_ahead = dayoffset - 1 - current_date.weekday() + 7
            return current_date + datetime.timedelta(days=days_ahead)
    elif interval == "2w":
        if current_date.weekday() <= dayoffset - 1:
            days_ahead = dayoffset - 1 - current_date.weekday()
            return current_date + datetime.timedelta(days=days_ahead)
        else:
            days_ahead = dayoffset - 1 - current_date.weekday() + 7
            return current_date + datetime.timedelta(days=days_ahead)
    elif interval == "1m":
        if current_date.day <= dayoffset:
            return datetime.date(current_date.year, current_date.month, dayoffset)
        else:
            return next_month_day(current_date, dayoffset)


def next_investment_date(pre_investment_date, dayoffset=0, interval="1d"):
    if interval == "1d":
        return pre_investment_date + datetime.timedelta(days=1)
    elif interval == "1w":
        return pre_investment_date + datetime.timedelta(days=7)
    elif interval == "2w":
        return pre_investment_date + datetime.timedelta(days=14)
    elif interval == "1m":
        return next_month_day(pre_investment_date, dayoffset)


class DCAStrategy(bt.Strategy):
    params = (
        ("investment_interval", "1m"),
        ("investment_dayoffset", 1),
        ("investment_amount", 100),  # Amount to invest each period
    )

    def __init__(self):
        self.total_invested = 0.0
        self.invested_amount = 0

        self.investment_date = None

    def next(self):
        current_date = self.data.datetime.date(0)
        if not self.investment_date:
            self.investment_date = start_investment_date(
                current_date,
                self.p.investment_dayoffset,
                self.p.investment_interval,
            )

        if current_date < self.investment_date:
            return

        price = self.data.close[0]
        size = self.p.investment_amount / price
        self.buy(size=size)

        self.total_invested += self.p.investment_amount
        self.investment_date = next_investment_date(
            self.investment_date,
            self.p.investment_dayoffset,
            interval=self.p.investment_interval,
        )

    def stop(self):
        title = strategy_title(self.p.investment_interval, self.p.investment_dayoffset)
        average_price = self.total_invested / self.position.size
        print(
            f"{title} | {self.p.investment_amount:.2f}| {self.total_invested:.2f} | {self.position.size:.2f} | {average_price:.2f}"
        )


@click.command()
@click.option("--symbol", default="BTC/USDT", help="交易对符号")
@click.option(
    "--interval",
    default="1m",
    type=click.Choice(["1d", "1w", "2w", "1m"]),
    help="投资间隔 (1d: 每日, 1w: 每周, 2w: 每两周, 1m: 每月)",
)
@click.option(
    "--dayoffset",
    default=1,
    type=int,
    help="每月投资日(1-31)或每周投资星期几(1-7)",
)
@click.option("--start-date", default="2020-01-01", help="开始日期 (YYYY-MM-DD格式)")
@click.option("--end-date", default="2024-12-31", help="结束日期 (YYYY-MM-DD格式)")
@click.option("--amount", default=60000, type=float, help="总投资金额")
def main(symbol, interval, dayoffset, start_date, end_date, amount):
    # Convert string dates to datetime objects
    start_date = datetime.datetime.strptime(start_date, "%Y-%m-%d")
    end_date = datetime.datetime.strptime(end_date, "%Y-%m-%d")

    cerebro = bt.Cerebro()

    data = download(symbol, start_date=start_date, end_date=end_date, interval="1d")

    data = bt.feeds.PandasData(dataname=data)  # pyright: ignore
    cerebro.adddata(data)

    cerebro.broker.setcash(2 * amount)
    cerebro.broker.setcommission(0.001)

    investment_amount = amount / investment_times(
        start_date, end_date, interval=interval
    )
    cerebro.addstrategy(
        DCAStrategy,
        investment_interval=interval,
        investment_amount=investment_amount,
        investment_dayoffset=dayoffset,
    )

    cerebro.run()


if __name__ == "__main__":
    main()
```

