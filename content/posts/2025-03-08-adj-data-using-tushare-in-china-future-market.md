---
title: "计算期货的复权数据"
date: 2025-03-08T17:16:49+08:00
draft: false
comment: true
description: "本文介绍如何计算期货的复权数据，以 Tushare 作为数据源。"
---

本周写点国内期货的内容，介绍如何通过 Python 计算期货主力连续合约的复权数据。

## 背景概述

我最近想测试下统计套利策略。因为希望合理分配资金、对冲风险，就计划测试下国内市场的表现，但 A 股不支持做空机制，于是转而选择了国内期货。

但问题是，期货不同于股票，不是一个品种一个可交易标的，而是由连续的交割合约组成，即每个合约都有到期日，而同一品种的不同月合约都是有价差的。

如下是玻璃2505和2509的合约的价格曲线：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-08-adj-data-using-tushare-in-china-future-market-02.png)

查看交易软件上的主合约价格图表，如下图所示是生猪的主力合约的价格图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-08-adj-data-using-tushare-in-china-future-market-03.png)

你会发现，这其中存在一些明显的跳空。这些跳空基本就是主连合约切换合约时产生的。

如果是短线交易或者跨期套利，这个问题或许不是那么重要。但我还有计划测试期货的 CTA 策略，还是要解决下这个问题。

如何解决？答案就是计算复权数据，通过复权计算买入持有的收益曲线。

熟悉股票交易的人都知道，交易软件或数据平台会提供股票的复权价格（如分红、拆股后的调整数据），以消除公司行为导致的历史价格断裂，确保长期收益计算的准确性。

但期货不同，交易软件和数据平台通常不会提供这个数据。

为什么呢？

所谓的主连合约是基于某个规则拼接而成，复权并非强制需求。根据策略目标不同，可自行选择换月规则，如固定日期、持仓量切换等。

散户能接触到的数据源通常只提供主连合约的原始价格数据。听过付费平台（如 Wind）好像有定制的主连合约，也提供了相应的复权因子。

那我这个小散该怎么办？

当然是自己计算了。走独立自主的发展道路，还可以根据策略目标制定不同的换仓时间和复权计算的方法。

开始前，先给大家看看，长持主连合约的收益与软件上主连合约价格曲线的差异。

以生猪为例：

```bash
python main.py --ts-code LH.DCE --method pre_close/pre_close --plot
```

绘制价格图如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-08-adj-data-using-tushare-in-china-future-market-04.png)

希望能助新手避坑。

上面使用的命令是我开发的一个脚本，可以直接将复权数据下载到本地。如果不想看完整文章，直接看文件代码地址：[download_future_adj_data.py](https://gist.github.com/poloxue/74fc77c6069a2293c6b776f0ea40a5bf)。这个代码是完整版，文章重点是介绍实现逻辑。

如下是这个命令的帮助信息：

```bash
Usage: main.py [OPTIONS]

Options:
  --ts-code TEXT    期货合约代码（例如：LH.DCE 表示生猪期货）  [必填]
  --start-date TEXT 起始日期（格式：YYYYMMDD，默认为20220101）

  --method          换月调整方法（默认：pre_close/pre_close）:
                    pre_close/pre_close: 使用前收盘价复权
                    open/pre_close: 使用开盘价/前收盘价复权
                    pre_settle/pre_settle: 使用前结算价复权
                    open/pre_settle: 使用开盘价复权

  --plot            是否显示价格曲线图（默认不显示）
  --help            Show this message and exit.
```

如果没有 Tushare 数据，文中也提供了关于如何自定义切换规则的一些思路和代码。或者联系我拿 Tushare 权限，有 20% 的折扣。我可以免费提供一个品种的复权数据。

---

正式进入主题，本文我将介绍如何基于 tushare 计算主连合约的复权数据。

核心有两部分：

- 一是拿到主连合约与实际合约的映射；
- 二是计算复权因子；

## 准备工作

下载安装将用到的几个 Python 库。

```bash
pip install tushare pandas matplotlib click tqdm
```

这里用到了一个不常用的库 - tqdm，它是用来展示数据下载进度的，特别当下载数量非常大时，这个小技巧能提升你的体验甚至效率。

简单示例：

```python
import time
from tqdm import tqdm

for _ in tqdm(range(100)):
    time.sleep(0.1)
```

效果如下：

```bash
100%|██████████████████████████| 100/100 [00:01<00:00, 86.14it/s]
```

导入所有库，配置认证 tushare 的 token。

```python
import tushare as ts
import pandas as pd
from tqdm import tqdm

ts.set_token("")
pro = ts.pro_api("Your Tushare Pro Token")
```

## 映射数据

映射数据从哪里拿？

我可以直接从数据提供商那里获取。如果能明确换仓规则，也可自己计算，即我知道如主连合约的底层合约何时从一个换到另一个。

### 标准映射数据

先说下如何拿到标准主连的映射关系，也就是如同花顺等交易软件上一样的主连合约规则，通过 tushare 的 `fut_mapping` 能直接拿到。我能接触到的数据源中，只在 tushare 看到了这个标准规则的映射关系。

如获取生猪的主连合约从2022年1月1日到现在与底层合约的映射，示例代码：

```python
mapping_data = pro.fut_mapping(
  ts_code="LH.DCE",
  start_date="20220101",
)
mapping_data.sort_values(
  by="trade_date",
  inplace=True,
  ignore_index=True,
)
print(mapping_data)
```

输出：

```bash
    ts_code trade_date mapping_ts_code
0    LH.DCE   20220104      LH2203.DCE
1    LH.DCE   20220105      LH2203.DCE
2    LH.DCE   20220106      LH2203.DCE
3    LH.DCE   20220107      LH2203.DCE
4    LH.DCE   20220110      LH2203.DCE
..      ...        ...             ...
762  LH.DCE   20250303      LH2505.DCE
763  LH.DCE   20250304      LH2505.DCE
764  LH.DCE   20250305      LH2505.DCE
765  LH.DCE   20250306      LH2505.DCE
766  LH.DCE   20250307      LH2505.DCE
```

如上的 `mappint_ts_code` 主连合约当前的底层合约。

我没有去深究这个规则，最常见的连续合约就是主力连续，主力合约似乎是以持仓为判断标准编织，还有其他的连续合约，如按固定日期切换，复权，最小价差等。

假设，Tushare 还提供了一个生猪连续合约的映射关系，在 `LH` 后加上 `L`，即 `LHL`，它的代码规则是 `主力合约代码+L`。如果想要所有的合约信息，可通过 tushare 的 `fut_basic` 接口拿到合约的基础信息。

### 自定义换仓规则

如果无法拿到这个标准的映射数据，可自定义一个尽量接近于标准的换仓规则。

我定义了一个固定日期换仓规则：

每月 28 号换仓到下一个合约，要求交割日期不少于 40 天。我将其封装成了一个函数，其中用到了 tushare 的 `fut_basic` 合约列表接口、`trade_cal` 交易日历接口。

函数定义：

```python
def fut_mapping(fut_code, exchange)
```

这是个简单例子，但在文章中介绍还是有点繁琐，就不展开了。请查看 [custom_fut_mapping.py](https://gist.github.com/poloxue/c3394991fa6dfb27bc2bce060378eef5)。

调用示例：

```python
mapping_data = fut_mapping("LH", exchange="DCE")
print(mapping_map)
```

输出：

```bash
      trade_date ts_code mapping_ts_code
0       20210108     LHC      LH2109.DCE
1       20210111     LHC      LH2109.DCE
2       20210112     LHC      LH2109.DCE
3       20210113     LHC      LH2109.DCE
4       20210114     LHC      LH2109.DCE
...          ...     ...             ...
1001    20250303     LHC      LH2507.DCE
1002    20250304     LHC      LH2507.DCE
1003    20250305     LHC      LH2507.DCE
1004    20250306     LHC      LH2507.DCE
1005    20250307     LHC      LH2507.DCE
```

其中的 `LHC` 是我随意定义的表示这个规则的生猪合约代码。

不知道这个代码有没有参考价值。

如果你没有途径拿到标准的主连合约的映射关系，这个或许可参考，将 tushare 替换为你的数据源即可。你可以修改规则，如按最大持仓，价差最小等等，计算这些规则所需的数据基本都是公开可获取的。

### 查询换仓详情

如现在我想查看换仓详情，只要记录上个交易日的 `mapping_ts_code`，比较前后 `mapping_ts_code` 是否相同。

```python
# 之前的合约
mapping_data["pre_mapping_ts_code"] = mapping_data["mapping_ts_code"].shift(1)
# 当前合约和之前合约不同且之前合约不为空
mapping_data["rollover"] = (
  (mapping_data["mapping_ts_code"] != mapping_data["pre_mapping_ts_code"]) &
  mapping_data["pre_mapping_ts_code"].notna()
)
print(mapping_data[mapping_data["rollover"]][
  ["ts_code", "trade_date", "mapping_ts_code", "pre_mapping_ts_code]
])
```

输出：

```bash
    ts_code trade_date mapping_ts_code pre_mapping_ts_code
20   LH.DCE   20220208      LH2205.DCE           LH2203.DCE
68   LH.DCE   20220419      LH2209.DCE           LH2205.DCE
143  LH.DCE   20220808      LH2301.DCE           LH2209.DCE
..      ...        ...             ...
667  LH.DCE   20241010      LH2501.DCE           LH2411.DCE
714  LH.DCE   20241216      LH2503.DCE           LH2501.DCE
747  LH.DCE   20250210      LH2505.DCE           LH2503.DCE
```

从上面就能看出来，合约是哪天（`trade_date`）从哪个合约（`pre_mapping_ts_code`）切换到当前的合约（`mapping_ts_code`）。

## 计算复权数据

到此，已经拿到了最难获取的主连合约映射关系，开始计算复权数据。

这块的核心是计算复权因子。有了复权因子，接着只要将复权因子作用到原始价格上即可拿到复权价格数据。

### 复权方法

常见的复权因子公式有哪些？

- open/pre_close，即当前合约的开盘价与上个合约收盘价的比值
- pre_close/pre_close，即当前合约和上个合约收盘价的比值；
- pre_settle/pre_settle，即当前合约与上个合约结算价的比值；
- open/open，即当前合约和上个合约的开盘价的比值；

如上统一采用的是比值计算方法，只是计算所用的价格不同。这篇文章会介绍前三种方法的实现，第四种 open/open 在脚本也实现了，就不再文章里展开了。

接下来具体介绍这四个复权因子的计算方法。

### open/pre_close

open/pre_close 是最简单的复权因子计算方法，因为在矩阵计算中，这最容易实现，不少文章就写了这种实现。对应到实际操作，即在合约切换收盘日平仓，次日在新合约重新开仓即可。

计算复权因子，要先拿到价格数据。

如果你用的 tushare 的映射关系，可以直接调用 `fut_daily` 拿到日线数据，通过 `open/pre_close` 即可。不过为了接下来其他计算方式的统一，我还是将实际合约的价格重新合并进来。

为了拿到实际合约的价格，我要知道每个合约在主连合约上出现的开始和结束时间。

```python
def extract_date_range(group):
    return group["trade_date"].min(), group["trade_date"].max()
date_ranges = mapping_data.groupby("mapping_ts_code").apply(extract_date_range)
print(date_ranges)
```

输出：

```bash
mapping_ts_code
LH2109.DCE    (20210108, 20210818)
LH2201.DCE    (20210819, 20211210)
LH2203.DCE    (20211213, 20220207)
...           ...
LH2501.DCE    (20241010, 20241213)
LH2503.DCE    (20241216, 20250207)
LH2505.DCE    (20250210, 20250307)
dtype: object
```

现在基于 `ts_code` 和 `date_range` 从 `fut_daily` 获取接口数据。

```python
all_ohlcvs = []
for ts_code, (start_date, end_date) in tqdm(date_ranges.items()):
    ohlcvs = pro.fut_daily(ts_code=ts_code, start_date=start_date, end_date=end_date)
    if not ohlcvs.empty:
        all_ohlcvs.append(ohlcvs)

ohlcv_data = pd.concat(all_ohlcvs, ignore_index=True)
```

将 `ohlcv_data` `mapping_data` 合并就能得到主连合约完整的数据了。

```python
data = mapping_data.merge(
    ohlcv_data,
    left_on=["mapping_ts_code", "trade_date"],
    right_on=["ts_code", "trade_date"],
    how="left",
    suffixes=("", "_1"),
)

data.drop(columns=["ts_code_1"], inplace=True)
print(data[["ts_code", "trade_date", "open", "pre_close_before"]])
```

输出：

```bash
    ts_code trade_date     open  pre_close
0    LH.DCE   20220104  14320.0    14450.0
1    LH.DCE   20220105  14165.0    14165.0
2    LH.DCE   20220106  14250.0    14245.0
3    LH.DCE   20220107  14150.0    14075.0
4    LH.DCE   20220110  13670.0    13915.0
..      ...        ...      ...        ...
762  LH.DCE   20250303  12950.0    12935.0
763  LH.DCE   20250304  13100.0    13125.0
764  LH.DCE   20250305  13160.0    13195.0
765  LH.DCE   20250306  13055.0    13070.0
766  LH.DCE   20250307  13100.0    13090.0
```

复权因子的计算是将 open/pre_close 即可，但这里还有有个小问题，默认的 `pre_close` 是当前映射合约上个交易日的收盘价，不是上个映射合约的上个交易日的收盘价。

一行代码解决：

```python
data["pre_close_before"] = data["close"].shift(1)
```

计算复权因子：

```python
# 换仓时的数据
rollover_data = data[data["rollover"]]
# 计算复权因子
data["rollover_factor"] = rollover_data["open"] / rollover_data["pre_close_before"]
data["adj_factor"] = data["rollover_factor"].fillna(1).cumprod()
```

计算复权价格并绘制收盘价：

```python
data["adj_close"] = data["close"] * data["adj_factor"]
data[["adj_close", "close"]].plot()
plt.show()
```

绘图如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-08-adj-data-using-tushare-in-china-future-market-01.png)

复权价格和实际的价格差异还是很大。

### pre_close/pre_close 和 pre_settle/pre_settle

pre_close/pre_close 和 pre_settle/pre_settle 的复权因子的计算也很简单了。因为这里是手动拼接的价格数据，当前合约和上个合约的价格上个交易日的价格都很容易拿到。

pre_close/pre_close：

```python
data["rollover_factor"] = rollover_data["prev_close"] / rollover_data["pre_close_before"]
data["adj_factor"] = data["rollover_factor"].fillna(1).cumprod()
```

pre_settle/pre_settle：

```python
data["pre_settle_before"] = data["settle"].shift(1)
data["rollover_factor"] = rollover_data["prev_settle"] / rollover_data["pre_settle_before"]
data["adj_factor"] = data["rollover_factor"].fillna(1).cumprod()
```

有了复权因子，就可以将价格数据与其相乘就能拿到复权价格了。

## 总结

这篇文章主要实现如何基于 tushare 计算期货的复权数据，包含了常见的复权计算方法，完整的脚本请查看地址：[download_future_adj_data.py](https://gist.github.com/poloxue/74fc77c6069a2293c6b776f0ea40a5bf)。如果没有 tushare 数据，文中也提供了实现的具体思路，即使没有合约映射数据，也可以自己制定规则实现。

希望本文对你有用。

---

最近有不少朋友希望我开群，如果有这个需求的，可以扫码加群：

<img src="https://cdn.jsdelivr.net/gh/poloxue/images@2025-03/2025-03-08-adj-data-using-tushare-in-china-future-market-05-v1.png" style="width:300px"><img>

补充：

<div style="color: #ff2020">

如果想直接用 Tushare 的数据，可联系我，有 20% 的折扣。Tushare 一年 200 元的积分等级就包含了这篇文章中的所有数据 API（这个等级其实已经支持了 tushare.pro 上 60% 的数据接口）。如只是临时需要，可以免费提供一个品种的复权数据。</div>
