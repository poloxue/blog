---
title: "开发了一个加密货币跨市场价差监控工具"
date: 2025-03-03T11:16:08+08:00
draft: false
comment: true
description: "我开发了一个加密货币跨交易所价差的监控工具。"
---

前两周发了一篇关于如何监控加密货币跨市场套利监控的文章，介绍了基础的实现思路和示例代码。有不少朋友对这个内容表现出了浓厚的兴趣，也有人想要这样一个监控工具。

上周我将示例代码重新整合，开发了一个终端监控工具，也修复了之前示例中的一些 bug，。如果还不是了解的，查看前文 [实时监控加密货币跨交易所的套利机会](https://www.poloxue.com/posts/2025-02-16-arbitrage-between-cryptocurrency-exchanges-using-ccxt/)。

## 使用示例

接下来，演示下在不同市场间的监控案例。提示：可使用 `Ctrl+q` 退出监控终端。

基于最新成交（Binance 现货和 OKX 正向永续合约的全量监控)：

```bash
$ python main.py --monitor-panel ticker \
                 --market-a binance.spot \
                 --market-b okx.swap.linear
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-03-cypto-spread-monitor-using-python-02.gif)

如上所示，默认展示的 USDT 合约，如果要切换成 USDC，可通过选项 `--quote-currency USDT` 实现。

而排序规则上，固定按价差百分比从大到小排序。

```bash
$ python main.py --monitor-panel ticker \
                 --market-a binance.spot \
                 --market-b okx.swap.linear \
                 --quote-currency USDC
```

如果是反向合约，如 `binance.swap.invese`，计价币种要切换成 USD，配置选项 `--quote-currency USD`。

当前版本的这个工具虽然是通过 asyncio 异步 IO 实现，但单进程的工具。全量监控时，，计算速度慢，实时性影响较大。如上图所示，延迟在秒级别。建议加上 --symbols 来明确监控范围，如 `--symbols BTC-USDT,ETH-USDT,XRP-USDT,TRUMP-USDT`。

```bash
$ python main.py --monitor-panel ticker \
                 --market-a binance.spot \
                 --market-b okx.swap.linear \
                 --symbols BTC-USDT,ETH-USDT
```

除了 ticker 价差监控，这个工具还是先了订单簿的价差监控，如下所示：

```bash
$ python main.py --monitor-panel orderbook \
                 --market-a binance.spot \
                 --market-b okx.swap.linear \
                 --symbols TRUMP-USDT,BTC-USDT,ETH-USDT,SOL-USDT,ADA-USDT,BNB-USDT,XRP-USDT
```

通过 `--monitor-panel` 切换监控类型为通过 orderbook 订单薄计算价差。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-03-cypto-spread-monitor-using-python-01.gif)

这个价差是基于挂单最佳买卖单计算而来。

挂单的消息数据量比 ticker 数据要高，如果全量监控的话，延迟能到 10秒以上的延迟。

## 源码下载

我已经将这个工具的代码放在了 Github 上，通过 Git 即可下载代码仓库。

命令如下：

```bash
$ git clone https://github.com/poloxue/seekopt
$ cd seekopt
$ pip install -r requirements.txt
```

说实话，我没有测试过 `requirements.txt` 中的依赖是否完整，如果遇到问题，可以在仓库 issue 中说明，或者加我微信。

## web 服务

这个监控工具的界面用的是 textual 开发的，textual 是一个 Tui 终端应用开发库。我暂时还不想整 Web 开发，就先用它实现了。

如果有意将其通过 web 访问，textual 也有命令可将其作为 web 服务。

```bash
$ textual serve "python main.py \
                                --market-a binance.swap.linear \
                                --market-b --bybit.swap.linear"
___ ____ _  _ ___ _  _ ____ _       ____ ____ ____ _  _ ____
 |  |___  \/   |  |  | |__| |    __ [__  |___ |__/ |  | |___
 |  |___ _/\_  |  |__| |  | |___    ___] |___ |  \  \/  |___ v1.1.1

Serving 'python main.py --market-a binance.swap.linear --market-b bybit.swap.linear'
on http://localhost:8000

Press Ctrl+C to quit
```

不过有个问题，这种方式提供的 web 服务，如果是多人使用，好像不是共享同一个监控脚本，这会导致资源的重复占用。

## 适用市场

这个工具是基于 ccxt 实现，按理说 ccxt 支持的市场都是支持的，不过我测试的时候，也发现了一些不适配的情况。我当前只测试了 binance、okx 和 bybit 三个交易所。

参数 market 格式是 `exchange.type.subtype`，支持类型如下所示：

```bash
exchange.spot
exchange.spot.margin
exchange.swap.linear
exchange.swap.inverse
exchagne.future.linear
exchagne.future.inverse
```

使用时，请把 exchange 替换为具体的交易所名称。

## 实时性

如果全量监控所有交易对，延迟会比较大，建议指定 symbols 限制监听的交易对数量。如果想提高实时性，最简单的方案多启动几个程序，按批次监听不同的交易列表。

我现在不确定这个工具是否确有需求，如果确有需求，我后续会考虑加入多进程和分布式能力，提升这个工具的实时性能。

## 延迟计算

关于这个监控工具上的实时性数值，是在计算价差时的时间减去消息上的时间戳得到。

这里有个问题，本地时钟和服务器时钟可能是不一样的。为了解决这个问题，程序会定时同步交易所的 servertime，计算与本地时间的偏差保持两边的同步，但获取服务器时间也有网络延迟，虽然说有粗略的方法计算这个延迟，但如果网络不好，有可能看到实时的数据是负数。

另外的方案，或许可以设置机器同步的 ntp server，保持和交易所时钟同步。这个或许可以尝试下。不过我没找到交易所用的是什么 ntp 服务器。

## 总结

这个价差监控工具还只是个初版实现。如果使用时遇到的什么问题或时有什么想法，请提出来，我会继续优化改进。

最后，希望本文对你有用。