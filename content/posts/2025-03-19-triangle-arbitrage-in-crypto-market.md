---
title: "实时监控加密货币现货市场的三角套利机会"
date: 2025-03-19T17:04:44+08:00
draft: false
comment: true
description: "本文继续介绍一个常见的套利策略，三角套利策略。"
---

本文继续介绍一个常见的套利策略，三角套利策略。我将介绍如何实时监控加密货币现货市场上的三角套利机会。

如果你看过我的前几篇文章，应该知道我最近一直专注在数字货币套利策略的学习研究。至于是否有低费率权限、硬件配置、低延迟网络等客观条件，不是文章的重点。我更多还是放在介绍套利的主线逻辑，完成我的整个套利系列的文章和工具集。请把它们当成一个抛砖引玉的 demo 即可，当你有这方面需求，能给你提供一些参考和思考，那就是好的。

正式开始本篇文章。

## 什么是三角套利？

三角套利原本是一种针对外汇市场的套利策略，通过利用三种货币对间的汇率失衡实现无风险利润。其核心原理基于“交叉汇率”一致性。

例如有 A/B、B/C、C/A 三种货币对，它们的实际价格与理论换算值有偏差，即按 A→B→C→A 兑换路径得到的 A 价值高于兑换前 A 的价值，我们即可进行三步循环交易，锁定汇率修正前的差价收益。

这个策略同样适用于数字货币市场。

如某交易所内的三个交易对 BTC/USDT、ETH/BTC 和 ETH/USDT，它们的价格分别是 30000、0.05 和 1520。假设无滑点且每次手续费为 0.1%。

现按路径 USDT→BTC→ETH→USDT 完成兑换。

- USDT -> BTC: 10000 USDT 买入 10000 / 30000 = 0.3333 BTC；
- BTC  -> ETH: 0.3333 BTC 兑换 0.3333 / 0.05 = 6.666 ETH；
- ETH -> USDT: 6.666 ETH 卖出 6.666 * 1520 = 10132 USDT；

三次交易即要扣除手续费（约 10000 * 0.001 * 3 = 30 USDT）后净收益约 132-30 = 103 USDT。

这是不是看似很美好。

实际上，随着加密货币市场逐渐成熟，这种纯理论套利机会也不是很多。且这种操作通常依赖自动化瞬时完成，需精确计算成本与收益。

## 实时监控工具

我实现了这样一个简单监控工具，看看当前市场是否存在这样的套利机会。

工具仓库地址 [github.com/poloxue/seekoptrader](https://github.com/poloxue/seekoptrader/tree/v0.0.2)


效果如下：

```bash
$ export PYTHONPATH=`pwd`
$ python seekoptrader/__main__.py triangle --exchange-name okx
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@seekoptrader/04.png)

如上图有六个汇率值，这是因为以不同币种作为起点和选择不同的兑换路径，一个三角配对可以有六种汇率计算方式。

假设你的费率是 0.1%，不考虑滑点的情况下，如果你能发现汇率大于 1.003 的机会，就有机会获利。如果是质押借贷，还要考虑借贷利息。

注：暂时不建议在 binance 上尝试，因为它的上面币种太多了，跑不动。不过或许它有机会的概率会高些。

## 实现思路

这个工具的实现和跨市场价差的监控类似，大致分三个步骤：

- 首先，匹配到可形成三角闭环交易对，毕竟希望监控能尽量覆盖全量市场；
- 其次，实时监控订单薄消息，这个和之前的跨市场类似，为了提高实时性，建议批量监控；
- 最后，计算三角汇率，我考虑了正反交易方向与不同计价币种的组合，将会计算出六组汇率；

与跨市场价差不同之处是，交易对的三角匹配和计算公式。

## 三角交易对匹配

三角套利的匹配比起跨交易所的匹配要复杂些，说下我遇到的有几个点吧。

### 匹配限制

首先，前面说三角套利是基于 A/B、B/C、C/A 这样的交易对组合实现闭环交易。加密货币市场上，基本都是按照 B/A（BTC/USDT）、C/B（ETH/BTC）、C/A（ETH/USDT）形成三角配对。

我不了解外汇市场的币对是什么规则，可能更加灵活吧。

为了便于确定汇率计算公式，我严格限制了交易对必按 B/A -> C/B -> C/A 格式和顺序的才能匹配成功。这样一来， 在确定基准单位（兑换起点）和兑换路径后，汇率公式就能确定下来了。不过这样不知道会不会溜掉一些交易对。

如 `A -> B -> C -> A` 的兑换路径：

```bash
1 / ask(B/A) / ask(C/B) * bid(C/A)
```

而 `B -> C -> A -> B` 的兑换路径：

```bash
1 / ask(C/B) * bid(C/A) / ask(B/A)
```

其中的 ask 和 bid 表示最优卖价和买价。


### 过滤规则

在用 ccxt 加载现货市场交易对时，发现了法币加密币的交易对，这些大概率无法交易，我直接过滤掉了。我梳理了几个主流交易所（binance、bybit、okx、bitget 和 coinbase）的法币，或许还有遗漏吧。

此外，我还设置了一个条件，过滤掉存在两个稳定币的三角，防止数量过于膨胀。基本思路是，稳定币的价格变化太小，资源有限的情况下，优先监控非稳定币。

### 代码片段

如下是加载配对某交易所可形成三角闭环交易对的代码：

```python
def find_triangles(self, markets):
    G = nx.DiGraph()
    for m in markets:
        base, quote, symbol = m["base"], m["quote"], m["symbol"]
        G.add_edge(base, quote, symbol=symbol)

    triangles = {}
    currencies = G.nodes()
    for b in currencies:
        for a in G[b]:
            for c in currencies:
                if c == b or c == a:
                    continue
                if c in G and b in G[c]:
                    if a in G[c]:
                        if self.valid_currencies([b, a, c]):
                            triangles[(a, b, c)] = (
                                G[b][a]["symbol"],
                                G[c][b]["symbol"],
                                G[c][a]["symbol"],
                            )
    return triangles
```

这里用了一个网络拓扑工具库 `networkx` 构建了一个有向图，从中搜索可形成闭环的三个交易对。其中的 `valid_currencies` 函数用于过滤一些配对，如法币或稳定币交易对。

## 兑换汇率计算

因为要更多可能监控交易机会，不限制兑换路径和计价币种。如何理解呢？

还是以 BTC/USDT、ETH/BTC 和 ETH/USDT 为例。

最容易想到的兑换路径是 USDT -> BTC -> ETH -> USDT，但也可以是 USDT -> ETH -> BTC -> USDT，这两个路径都是以 USDT 为计价币。我还可以从 ETH 为基点，即 ETH 计价，那么兑换路径就是 ETH -> USDT -> BTC -> ETH，或是 ETH -> BTC -> USDT -> ETH。

一旦路径和基点变了，那计算的公式也就完全不同了。

如 `USDT -> BTC -> ETH -> USDT` 的兑换路径，公式是：

```bash
1 / ask(BTC/USDT) / ask(ETH/BTC) * bid(ETH/USDT)
```

而 `BTC -> ETH -> USDT -> BTC` 的兑换路径，公式是：

```bash
1 / ask(ETH/BTC) * bid(ETH/USDT) / ask(BTC/USDT)
```

这样组合起来，一个三角配对就有了 6 个交易路径，这也是为什么上面演示的工具中有 6 个 汇率的原因。

现在很多币种都可以质押借款，如果能抓住这瞬时的交易机会，在还掉借贷后，还能赚点币在手里，这就是利润。

## 灵活推算汇率公式

除了前面按交易对模式固定死汇率公式，还可以更灵活的推算公式，只要提供兑换起点币种和三角交易对，就能推出三角兑换的汇率公式。

我已经把这个思路的核心逻辑代码写出了，借这篇文章保存下，万一哪天需要还能找到。

```python
from collections import defaultdict


def detect_arbitrage_ops(start_currency, symbols):
    graph = defaultdict(dict)
    for symbol in symbols:
        base, quote = symbol.split("/")
        graph[base][quote] = symbol  # 正向交易 base→quote
        graph[quote][base] = symbol  # 反向交易 quote→base

    chains = []
    max_depth = 3

    def dfs(current, path, ops=[], depth=0):
        if depth == max_depth:
            if current == start_currency:
                chains.append(ops)
            return

        for next_currency in graph[current]:
            symbol = graph[current][next_currency]
            base, quote = symbol.split("/")
            new_op = f"*bid({symbol})" if current == base else f"/ask({symbol})"
            dfs(next_currency, path + [next_currency], ops + [new_op], depth + 1)

    dfs(start_currency, [start_currency])

    return ["1" + "".join(chain) for chain in chains]
```


示例测试:
```python
base = "ETH"
pairs = ["BTC/USDT", "ETH/BTC", "ETH/USDT"]
operations = detect_arbitrage_ops(base, pairs)
print("ETH 兑换起点:", operations)
```

输出：

```bash
ETH 兑换起点: [
    '1*bid(ETH/BTC)*bid(BTC/USDT)/ask(ETH/USDT)',
    '1*bid(ETH/USDT)/ask(BTC/USDT)/ask(ETH/BTC)'
]
```

现在的已经简单够用，就没把这个设计集成到工具里。


## 总结

本文简单介绍了三角套利，开发了一个监控小工具。这是我的一次尝试，如有建议请提出来。如果你对代码实现感兴趣，可查看：[triangle/monitor.py](https://github.com/poloxue/seekoptrader/blob/v0.0.2/seekoptrader/arbitrage/triangle/monitor.py)。

最后，希望本文对你有用。

最近感冒不舒服，脑子有点不清楚，希望文中不要有太多错误。
