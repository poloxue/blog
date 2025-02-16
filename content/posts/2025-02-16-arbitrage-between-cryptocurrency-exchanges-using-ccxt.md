---
title: "实时监控加密货币跨交易所的套利机会"
date: 2025-02-16T17:18:36+08:00
draft: false
comment: true
description: "有评论希望我展开来说说跨交易所的套利机会。一直觉得随着参与这类策略的人多了，机会很少了，就没有深入研究过。这次借这个机会尝试一下。"
---

前面写了一篇关于[如何用 Python 获取数字货币行情](https://www.poloxue.com/posts/2025-02-10-crypto-currency-market-data-using-ccxt/)的博文，有评论希望我展开来说说跨交易所的套利机会。一直觉得随着参与这类策略的人多了，机会很少了，就没有深入研究过。这次借这个机会尝试一下。

本文是我的一次尝试，将介绍如何实现一个跨交易所套利机会的监控工具。

<p style="color:red">免责声明：本文内容仅供学习研究使用，不构成任何投资或交易建议。实际应用前请自行评估风险，作者不对由此产生的损失承担责任。</p>

## 是什么？

什么是跨交易所的价差套利？跨交易所套利是通过捕捉同一资产在不同交易所的价格差异实现低风险收益的策略。

例如，当 BTC 在 Binance 价格为 90000，而在 OKX 为 90200 时，就可以在 Binance 低价买入并在 OKX 高价卖出。当它们的价格重新回归到相等时，平掉仓位，即可得到价差带来的收益。

价差套利的分类有很多，如期现套利、跨期套利、跨品种套利，价差套利其实就是将相关性的品种进行对冲交易。这些套利策略中，有些是低风险套利，而有些可能存在较大风险，不同品种间的相关性有可能失效，或者短期出现超越风险承受力的大幅度波动。

本文介绍是加密货币跨交易所套利，尝试开发一个工具监控不同交易所间的相同币种的实时价格发现潜在的套利机会。

再次提示：这是我的一次尝试而已，和金融交易相关的内容还是要小心。

## 实现路径

在开发这个工具前，我要先明确工具实现的流程，大概分几个步骤。

- 筛选出跨交易所间可配对品种，且要过滤或修正一些错配；
- 监听筛选出的品种的最近价格，计算其配对价差；
- 从应用角度分析，如何将这个监控结果运用于实际交易中；

开始上手实操吧！

## 配对品种

开始第一步部分，写如何将不同交易所的交易对匹配起来。

如何识别呢？

我将通过代码拉取两个交易所的交易对，按计价和基准币种配对，如 BTC/USDT，计价货币是 USDT、基准货币是 BTC。

还有，配对时要明确品种类型，避免不符合预期的配对，如希望配对交易所 A 和 B 的永续合约 ，但配对的却是 A 交易所的现货和 B 交易所的永续合约。

CEX 的品种类型，有主类型和子类型的区别，主类型可能就是现货 spot、永续合约 swap、交割合约 future、期权 option，而子类型如对合约，无论是永续还是交割，都有正向合约（U本位）、反向合约（币币本位）。这块我们不用考虑 option，它和其他交易品种差异比较大。

如果没接触这部分内容，可能难以理解的，其实画一张关系图画，就容易理解了。但是我有点懒。

我希望这个匹配的函数的定义如下：

```python
def load_pairs(
    exchange_a, # 交易所 a 实例
    exchange_b, # 交易所 b 实例
    type_a = "spot", # 品种 a 注类型
    subtype_a = None, # 品种 a 子类型
    type_b = "spot", # 品种 b 注类型
    subtype_b = None, # 品种 b 子类型
)
```

详细的实现代码，如下所示：

```python
import os
import ccxt

params = {
    'enableRateLimit': True,
}

def load_pairs(
    exchange_a,
    exchange_b,
    type_a = "spot",
    subtype_a = None,
    type_b = "spot",
    subtype_b = None,
):
    exchange_a.load_markets()
    exchange_b.load_markets()

    markets_a = {
        (m['base'], m['quote']): m['symbol']
        for m in exchange_a.markets.values() 
        if m['type'] == type_a and (subtype_a is None or m[subtype_a])
    }
    markets_b = {
        (m['base'], m['quote']): m['symbol']
        for m in exchange_b.markets.values()
        if m['type'] == type_b and (subtype_b is None or [subtype_b])
    }

    pair_keys = set(markets_a.keys()).intersection(set(markets_b.keys()))
    return [
        {
            'base': base,
            'quote': quote,
            'symbol_a': markets_a[(base, quote)],
            'symbol_b': markets_b[(base, quote)],
        }
        for base, quote in pair_keys
    ]
```

如下示例代码，将不同交易所的交易对关联起来的。

```python
binance = ccxt.binance(params)
okx = ccxt.okx(params)
# Binance 现货对 OKX 现货
pairs = load_pairs(binance, okx, type_a='spot', type_b='spot')
# Binance 现货对 OKX 正向永续合约
pairs = load_pairs(binance, okx, type_a='spot', type_b='swap', subtype_b='linear')
```

同一个交易所的不同类型品种当然也是可以配对的。

```python
# 初始化交易所对象
okx = ccxt.okx(params)
# OKX 现货对 OKX 正向永续合约
pairs = load_pairs(okx, okx, type_a='spot', type_b='swap', subtype_b='linear')
```

不过即使按照上述的规则，依然会出现错配。为什么？

因为不同交易所可能出现同名不同币的情况，如 NEIRO 在 OKX 和 Bybit 就是不同币种，OKX 上有两个 NEIRO，其中和 Bybit 配对的是 NEIROETHUSDT 合约。具体原因就不介绍了。

为了防止这类情况，可以将价差异常的配对找出，人工确认原因。

尝试实现这个代码：

```python
def detect_abnormal_pairs(exchange_a, exchange_b, pairs, threshold = 0.05):
    """
    用于检测价差异常的函数

    参数说明：
    :param matched_pairs: load_pairs函数返回的匹配交易对列表
    :param threshold: 视为异常的价格差异比例（0.05表示5%）
    
    返回结构：
    {
        'base': 基准货币,
        'quote': 计价货币,
        'symbol_a': 交易所A交易对,
        'symbol_b': 交易所B交易对,
        'price_a': 原始价格A,
        'price_b': 原始价格B,
        'spread_ratio': 价差比例,
        'is_abnormal': 是否异常
    }
    """
    normal_pairs = []
    abonormal_pairs = []
    for pair in pairs:
        try:
            # 获取最新行情数据（单次尝试）
            ticker_a = exchange_a.fetch_ticker(pair['symbol_a'])
            ticker_b = exchange_b.fetch_ticker(pair['symbol_b'])

            # 获取最后成交价
            price_a = ticker_a.get('last')
            price_b = ticker_b.get('last')

            # 跳过无效价格
            if None in [price_a, price_b]:
                print(f"价格缺失: {pair['symbol_a']}/{pair['symbol_b']}")
                continue

            # 计算价差比例（基于较小价格）
            min_price = min(price_a, price_b)
            spread_ratio = abs(price_a - price_b) / min_price
 
            # 构建结果对象
            result = {
                **pair,
                'price_a': price_a,
                'price_b': price_b,
                'spread_ratio': spread_ratio,
                'is_abnormal': spread_ratio > threshold
            }

            if result['is_abnormal']:
                abonormal_pairs.append(result)
            else:
                normal_pairs.append(pair)
        except Exception as e:
            print(f"处理交易对 {pair} 时发生错误: {str(e)}")

    return abonormal_pairs, normal_pairs
```

为了以防万一，异常的阈值可设置的小一些，如 0.05 即可。不过，期现的价差是有可能大于这个值的。

如下代码将匹配交易对保存到 csv 文件。

```python
binance = ccxt.binance(params)
okx = ccxt.okx(params)

pairs = load_pairs(binance, okx, 'spot', 'future')
abnormal_pairs, normal_pairs = detect_abnormal_pairs(binance, okx, pairs, 0.05)
pd.DataFrame(normal_pairs).to_csv("normal_pairs.csv")
pd.DataFrame(abnormal_pairs).to_csv("abnormal_pairs.csv")
```

得到这两个 csv 文件后，建议手动接入检查文件中的配对情况，确认无误。

还有一个坑，就算是相同币种，这个脚本也可能漏掉它们，如 PEPE 永续合约，在 OKX 的价格和 Binance 是不同的，因为 Binance 对 PEPE 放大了 1000 倍，名为 1000PEPEUSDT，基准货币是 1000PEPE，而不是 PEPE。这个特殊逻辑我就不实现了，有兴趣自行研究。通常都是一些 meme 币会出现这种情况。

当然，如果你不是想全量监控，而是清楚想监控的是什么配对交易对，就无需全量匹配了。

## 实时监控价差

接下来就是要监控它们的价差了。为了实时监控，还是通过 ccxt.pro 的 `watch_ticker` 方法实时监听最新成交价计算价差。

最简单的方式就是顺序监控。

```python
ticker_a = await exchange_a.watch_ticker(symbol_a)
ticker_b = await exchange_b.watch_ticker(symbol_b)
spread_pct = abs(ticker_a['last'] - ticker['last'])/min(ticker_b['last'])
```

不过为了更加实时，去掉两者等待的依赖，我用 `watch_tickers` 分别监听交易所 a 和 b 的所有 symbol，监听到新的价格就立刻计算价差。

稍微有点复杂，不想展开了说明，直接看代码吧。

```python
import os
import asyncio
import ccxt.pro as ccxtpro
import pandas as pd
import traceback
from typing import Dict, Tuple
from collections import defaultdict

params = {
    'enableRateLimit': True,
    # 'proxies': {
    #     'http': os.getenv('http_proxy'),
    #     'https': os.getenv('https_proxy'),
    # },
    # 'aiohttp_proxy': os.getenv('http_proxy'),
    # 'ws_proxy': os.getenv('http_proxy')
}

class Monitor:
    def __init__(self, exchange_a, exchange_b, pairs):
        self.exchange_a = exchange_a
        self.exchange_b = exchange_b
        
        # 构建symbol映射关系
        self.symbol_map = self._build_symbol_map(pairs)
        self.pair_data: Dict[Tuple[str, str], dict] = {}
        self.monitor_tasks = []
        self.running = False

    def _build_symbol_map(self, pairs):
        """构建symbol到配对关系的快速映射"""
        symbol_map = defaultdict(dict)
        for pair in pairs:
            key = (pair['base'], pair['quote'])
            symbol_map['a'][pair['symbol_a']] = {'index': 'a', 'pair_key': key}
            symbol_map['b'][pair['symbol_b']] = {'index': 'b', 'pair_key': key}
        return symbol_map

    async def monitor(self, exchange, index: str):
        """
        统一监控方法
        :param exchange: 交易所实例
        :param index: 来源索引 ('a'或'b')
        """
        symbols = [s for s in self.symbol_map[index]]

        while self.running:
            try:
                tickers = await exchange.watch_tickers(symbols)
                await self.process_tickers(tickers, index)
            except Exception as e:
                traceback.print_exc()
                print(f"监控异常({index}): {str(e)}")
                await asyncio.sleep(5)

    async def process_tickers(self, tickers, index):
        """处理批量ticker数据"""
        for symbol, ticker in tickers.items():
            if symbol not in self.symbol_map[index]:
                continue

            pair_map = self.symbol_map[index][symbol]
            pair_key = pair_map['pair_key']
            
            # 初始化数据结构
            if pair_key not in self.pair_data:
                self.pair_data[pair_key] = {
                    'price_a': None,
                    'price_b': None,
                    'spread': None
                } 

            price_field = f'price_{index}'
            self.pair_data[pair_key][price_field] = ticker['last']
 
            # 立即计算价差
            await self.calculate_spread(pair_key)

    async def calculate_spread(self, pair_key):
        """带校验的价差计算"""
        data = self.pair_data[pair_key]
        try:
            if data['price_a'] and data['price_b']:
                min_price = min(data['price_a'], data['price_b'])
                spread_pct = abs(data['price_a'] - data['price_b']) / min_price
                data['spread_pct'] = spread

                # 触发报警的价差阈值
                if spread > 0.01:  
                    await self.trigger_arbitrage(pair_key)
        except (TypeError, ZeroDivisionError) as e:
            print(f"价差计算错误 {pair_key}: {str(e)}")

    async def trigger_arbitrage(self, pair_key):
        """触发套利逻辑"""
        data = self.pair_data[pair_key]
        print(f"套利机会! {pair_key} 价差: {data['spread']:.2%}")
        # 此处添加实际交易逻辑

    def start(self):
        """启动监控任务"""
        self.running = True
        self.monitor_tasks = [
            asyncio.create_task(self.monitor(self.exchange_a, 'a')),
            asyncio.create_task(self.monitor(self.exchange_b, 'b'))
        ]

    async def stop(self):
        """优雅关闭"""
        self.running = False
        await asyncio.gather(*self.monitor_tasks, return_exceptions=True)
        await self.exchange_a.cloase()
        await self.exchange_b.cloase()
```

上面实现了一个 `Monitor` 类，只要监听一个新的价格，都会重新计算 spread 价差，评估是否触发报警或者进入到是否交易的验证中。

如下代码使用这个类监控价差：

```python
async def main(exchange_a, exchange_b, pairs):
    await exchange_a.load_markets()
    await exchange_b.load_markets()

    monitor = Monitor(exchange_a, exchange_b, pairs)
    monitor.start()
    
    try:
        while True:
            await asyncio.sleep(1)
            # 可在此处可添加其他逻辑：
    except KeyboardInterrupt:
        await monitor.stop()
        print("监控已停止")

if __name__ == "__main__":
    try:
        pairs_df = pd.read_csv("pairs.csv")
        pairs_dif = pairs_df[pairs_df['quote'] == "USDT"]
        required_cols = ['base', 'quote', 'symbol_a', 'symbol_b']
        if not all(col in pairs_df.columns for col in required_cols):
            raise ValueError("CSV文件缺少必要列")
        pairs = pairs_df[required_cols].to_dict('records')
    except Exception as e:
        print(f"配置加载失败: {str(e)}")
        exit(1)

    # 交易所初始化
    exchange_a = ccxtpro.binance(params)
    exchange_b = ccxtpro.okx(params)
    
    try:
        asyncio.run(main(exchange_a, exchange_b, pairs))
    except KeyboardInterrupt:
        print("程序已终止")
```


## 如何应用？

监控得到价差数据后，怎么用呢？

没有成熟的自动化交易策略的话，可以先做个报警消息观察观察，或者基于它做个实时排序，通过页面实时监控这些品种的价差信息。

用于自动化交易的话肯定有风险的，实盘要考虑问题比较多，价差和实际的交易是有区别的，真实交易肯定要看 order book 的买卖价，对于流动性不足的品种，最近成交价和实际能交易的价格可能会相差很大，滑点很大。

价差监控最好是用 orderbook 的买卖价和深度。我这里用最新价是因为监听 ticker 的数据量会小很多，order book 实时数据频率非常高，监控太多品种的负载太高。

真实交易，除了滑点，还有交易费用，对于保证金交易可能还有如借贷利息、资金费率等成本，都是要考虑的。如果不解决这些问题，得到的可能就是不断亏损的套利。

在成熟的市场，我觉得普通人是很难有这种套利机会的，不过加密货币市场，猜测小币的套利机会应该还是有的。还有就是，这个市场每隔个一两月就会有一次异常的暴跌，这或许是价差套利的最佳时机，当然也可能会爆仓。

再次声明，这是我的一次研究尝试，如要在真实场景下使用，请自行研究风险。

最后，希望本文对你有用。