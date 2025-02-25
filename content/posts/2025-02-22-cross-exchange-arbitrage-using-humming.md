---
title: "通过 Hummingbot 运行一个做市机器人"
date: 2025-02-22T16:21:45+08:00
draft: false
comment: true
description: "最近在研究了下一个叫 Hummingbot 的 Python 加密货币机器人，它其中提供了高频做市、跨市场套利和资金费率套利等策略。"
---

最近研究了一个叫 Hummingbot 的 Python 加密货币机器人，它提供了一些常见的高频策略，如高频做市、跨市场套利和资金费率套利等。我很早就知道 Hummingbot，但一直没有测试它，主要是它用起来不舒服，代码不是很清晰。

Hummingbot 是一款高频量化机器人，里面有大量 C++ 代码，还有基本都是 asyncio 异步操作。我决定搞 Hummingbot 也是因为它内置了一些高频做市、跨市场套利、资金费率套利和 cex/dex 套利交易等策略，作为高频交易的学习材料，还是不错的。还有，最近发现它还支持 CEX/DEX 套利，那无论如何也要搞下它了。

事件机制的话，Hummingbot 中的 `on_tick` 不是由成交 tick 触发的，而是定时的 clock tick，如配置 1 秒、1分钟定时触发。更细粒度的时间控制，最小可设置到 0.1s。这个和 vnpy 不同，我好像记得之前有人吐槽 vn.py 不支持定时触发的能力，因此更喜欢 backtrader。

Hummingbot 的数据源基本都是 ws 实时监听得到，频繁的的获取数据，如价格、订单簿（OrderBook），不会触发交易所的频率限制。当然，下单是例外，这是主动触发的操作。

本文尝试走一遍流程，跑一个它的内置做市机器人试试。

## 交互模式

Hummingbot 提供了两种交互模式 Tui 终端和 Web 端的 Dashboard。早期版本只提供了 TUI 的终端交互模式，完全是通过命令行管理的机器人。

2023 年上线 Web 端的 Dashboard，提供了 Web 端管理机器人的能力。Dashboard 是基于 streamlit 实现的。前段时间，我还用 streamlit 做了个简单的 A 股股票筛选器 demo，有兴趣移步到 [使用 Streamlit 打造一个股票筛选分析工具](https://www.poloxue.com/posts/2025-01-13-create-a-stock-screener-using-streamlit/)。

本文，我将重点通过 Tui 终端模式演示如何运行 Hummingbot 的机器人。

## 安装启动

Hummingbot 的安装可通过源码或是 docker。如果不是要阅读源码或调试问题，建议是用 docker 安装，毕竟简单省力，把更多精力到核心功能上。当然，源码安装也不复杂，但要有好的网络。

> 注：开始前，请确保你已经安装 Docker 了。

### 下载代码仓库

使用 git 下载 Hummingbot 的代码仓库。

```bash
git clone https://github.com/hummingbot/hummingbot.git
```

我在下载的时候常遇到网络 Timeout，可通过 --depth=1 只下载最新的文件，提升下载速度。

```bash
git clone --depth=1 https://github.com/hummingbot/hummingbot.git
```

### 启动应用

进入 `hummingbot` 目录通过 `docker` 启动服务：

```bash
docker compose up -d
```

现在，用 `docker attach` 命令就能进到应用入口：

```bash
docker attach hummingbot
```

你会看到如下画面。

![](https://hummingbot.org/assets/img/welcome.png)

第一次启动，会要求你设置登陆密码，以后的每次进入 Hummingbot 终端都离不开这个密码。

补充一点，`exit` 退出交互模式时，整个服务都会停止，要重新 `docker-compose up -d` 启动，`docker attach` 重新登陆。如果想退出但保持运行状态，可通过 `Ctrl+p` + `Ctrl+q` 快捷键组合（保持 `Ctrl` 按住状态）即可退出。

### 查看帮助信息

成功登陆后，你将会进入到一个 Hummingbot 的终端，如果想查看它支持什么命令，输入 `help` 查看它支持的命令，这些命令提供了在终端管理机器人的能力。

```bash
> help
 usage: {connect,create,import,help,balance,config,start,stop,status,history,gateway
  ,exit,export,ticker,previous,mqtt,spreads,rate,order_book,tab_example} ...

  positional arguments:
    {connect,create,import,help,balance,config,start,stop,status,history,gateway,exit
  ,export,ticker,previous,mqtt,spreads,rate,order_book,tab_example}
      connect             List available exchanges and add API keys to them
      create              Create a new bot
      import              Import an existing bot by loading the configuration file
      help                List available commands
      balance             Display your asset balances across all connected exchanges
      config              Display the current bot's configuration
      start               Start the current bot
      stop                Stop the current bot
      status              Get the market status of the current bot
      history             See the past performance of the current bot
      gateway             Helper comands for Gateway server.
      exit                Exit and cancel all outstanding orders
      export              Export secure information
      ticker              Show market ticker of current order book
      previous            Imports the last strategy used
      mqtt                Manage MQTT Bridge to Message brokers
      spreads             Set bid and ask spread
      rate                Show rate of a given trading pair
      order_book          Display current order book
      tab_example         Display hello world
```

我将重点用到几个命令：

- `connect`，用于配置交易所连接器；
- `balance`，查询账户余额；
- `create`，创建机器人配置；
- `start`，运行机器人；
- `stop`，停止当前的机器人；
- `status`，监控机器人运行状态；
- `history`，查看机器人的历史表现；
- `config`，配置全局参数；

到此，就完成了 Hummingbot 基础安装。


## 实现步骤

正式开始前，先介绍下大概的实现流程，主要分为如下的四个步骤。

1. 连接交易所，介绍如何配置和连接交易所，查看账号的信息，如当前的账户余额；
2. 配置启动策略，介绍配置并成功一个 Hummingbot 内置的简单做市的策略机器人；
3. 监控运行状态，介绍如何监控运行中机器人的状态，如订单、成交、盈亏和行情等；

接下来，开始正式行动吧！

## 连接交易所

这部分会涉及两个执行步骤。首先要确认用什么交易所，其次是配置交易所的认证信息和连接。

### 选择交易所

选择交易所前，先看下 Hummingbot 支持的交易所列表吧。输入 `connect`，列出可用的交易所，输出如下：

```bash
> connect
Testing connections, please wait...
+---------------------------+------------+-----------------+
| Exchange                  | Keys Added |  Keys Confirmed |
|---------------------------+------------+-----------------|
| binance                   | No         | No              |
| binance_perpetual         | No         | No              |
| binance_perpetual_testnet | Yes        | Yes             |
| binance_us                | No         | No              |
| bitget_perpetual          | No         | No              |
| bybit                     | No         | No              |
| bybit_perpetual           | No         | No              |
| bybit_perpetual_testnet   | No         | No              |
| bybit_testnet             | No         | No              |
| dydx_v4_perpetual         | No         | No              |
| okx                       | No         | No              |
| okx_perpetual             | No         | No              |
+---------------------------+------------+-----------------+
```

这里我只保留了一些常用的交易所，你如果执行命令会看到更多的选项。

从上面可知，我已经设置了 Binance 合约的 testnet 环境，所以 `binance_perpetual_testnet` 的 `Key Added` 和 `Keys Confirmed` 都是 `Yes`。其他如常用的 `okx` 和 `binance` 是都是支持的，那就是它们了。

连接器名称上 `exchange` 和 `exchange_perpetual` 的区别就是现货和合约模式的区别。如果希望支持做空，就用支持合约的交易连接器，OKX 就是 `okx_perpetual`，而 Binance 就是 `binance_perpetual`。否则就是 `okx` 和 `binance`。

### 配置交易所

接下来开始配置这两个连接器，要拿到 API 的认证信息，如 `API_KEY` `API_SECRET`，OKX 还配置 `PASSPHRASE`。拿到 API 认证信息后，通过 `connect exchange` 即可将认证信息配置到系统。

如下命令：

```bash
> connect okx
> connect okx_perpetual
> connect binance_perpetual
```

按照 Hummingbot 终端的的交互提示配置就行了，一点也不复杂。

这里吐槽两句，我本来想用 bybit 来测试的，在添加认证信息时，一直提示错误，查看 Hummingbot 的官方 issue，发现似乎修复代码还在 `development` 分支。

特别提醒，Hummingbot 支持的是基础账户模式，如果升级了统一账户，有可能就不支持了。Binance 可通过开子账号解决这个问题的，OKX 的话， 将账户模式切换回基础模式下的 "现货合约模式" 即可。

### 查询余额

连接成功后，可以查下账户的余额信息，看看是不是真的可用状态。

```bash
> balance
  Updating balances, please wait...

  binance_perpetual:
      Asset   Total Total ($) Allocated
       USDT 27.1375     27.13        0%
    Total: $ 27.13
  Allocated: 0.00%

  okx:
      Asset   Total Total ($) Allocated
       USDT 29.9658     29.96        0%
    Total: $ 29.96
  Allocated: 0.00%

  okx_perpetual:
      Asset   Total Total ($) Allocated
       USDT 29.9658     29.96        0%
    Total: $ 29.96
  Allocated: 0.00%

  Exchanges Total: $ 87
```

接下来开始启动一个真正的交易机器人了。

## 配置运行策略

配置和运行策略这部分的花了我不少时间，有些问题还是要阅读代码才能确认问题所在。

这里虽然只跑了一个策略，但其实为了把它的架构搞清楚，测试了好几个策略，包括之前的 v1 版本机器人。哦，对了，现在我要介绍的策略是 v2 版本，下面介绍的都是 v2 版本策略的配置方式

### 策略模版

Hummingbot 默认提供了一些策略模版，执行 `create --script-config` 会显示可用模版。

```bash
create --script-config [如下是输入补全提示]
                        simple_pmm
                        simple_vwap
                        simple_xemm
                        v2_directional_rsi
                        v2_funding_rate_arb
                        v2_twap_multiple_pairs
                        v2_with_controllers
                        v2_xemm
```

如下这几个常见策略的简单说明:

- **基础做市（simple_pmm）**：通过持续挂单捕捉买卖价差，适用于主流交易对的流动性维护。 
- **跨交易所做市（simple_xemm/v2_xemm）**：在一家交易所挂单，另一家对冲风险，利用交易所间价差获利。
- **时间加权（v2_twap_multiple_pairs）**：将大额订单拆分为多笔小额订单，按时间均匀执行以减少市场冲击，支持多交易对。  
- **成交量加权（simple_vwap）**：根据市场实时成交量动态调整订单规模，降低交易滑点。
- **方向性 RSI（v2_directional_rsi）**：基于 RSI 指标判断超买超卖信号，触发趋势跟踪交易。  
- **资金费率套利（v2_funding_rate_arb）**：通过永续合约与现货间的资金费率差异进行对冲套利。
- **控制器增强（v2_with_controllers）**：这个比较复杂，可理解为总控管理多个子策略，总控负责风险控制、组合管理等。

我就那个 `simple_pmm` 先演示如何在 Hummingbot 中创建一个机器人吧！

### 配置策略

在 `simple_pmm` 配置时，有两个注意点：

首先，我在 `okx_perpetual` 上测试 `simple_pmm` 这个做市策略，发现它只适用于现货，不支持合约。

还有，`simple_pmm` 是在订单簿上下挂单，而现货不同于合约，保证交易对两个币种钱包都有钱才能成功挂单。

有了这些前提，开始尝试具体的配置吧！

首先，通过 `create --script-config` 命令配置脚本参数，按交互式提示输入你的参数即可。

```bash
>>>  create --script-config simple_pmm

  Exchange where the bot will trade >>> okx
  Trading pair in which the bot will place orders >>> SOL-USDT
  Order amount (denominated in base asset) >>> 0.01
  Bid order spread (in percent) >>> 0.003
  Ask order spread (in percent) >>> 0.003
  Order refresh time (in seconds) >>> 100
  Price type to use (mid or last) >>> mid
  Enter a new file name for your configuration >>> conf_simple_pmm_sol.yml
  A new config file has been created: conf_simple_pmm_sol.yml
```

这个配置的意思是，在 order book 中间价上下 0.3% 的位置分别挂上 0.01 SOL 的买卖订单，每 60 秒会刷新重新下单。

最后，将这个参数保存到命名为 `conf_simple_pmm_sol.yml` 的配置文件中。

补充一点，如果在配置参数时，遇到配置错误，希望重新配置，可通过 `Ctrl+x` 退出参数配置模式，重新进入。

### 启动策略

通过 `start` 启动这个策略机器人，格式是 `start --script 策略脚本 --conf 策略配置文件`，即用什么参数运行什么策略。

命令如下：

```bash
> start --script simple_pmm.py --conf conf_simple_pmm_sol.yml
```

如果一切配置正常，查看你的活跃订单，就能看到成功下了两个订单了。

或者执行 `status` 命令，查看机器人当前的运行状态： 

```bash
> status
    Balances:
      Exchange Asset  Total Balance  Available Balance                                    
           okx   SOL       0.079768           0.079768                                    
           okx  USDT    18.86025163        17.49605163

    Orders:
      Exchange   Market Side     Price  Amount      Age
           okx SOL-USDT  buy 136.42161    0.01 00:00:38
```

如果加上 `--live` 选项，可实时监控机器人。


### 策略表现

如果想查看策略的表现，执行 `history` 查看，输出如下：

```bash
> history

Start Time: 2025-02-25 14:26:14
Current Time: 2025-02-25 15:24:17
Duration: 0 days 00:58:02

okx / SOL-USDT

  Trades:
                                   buy    sell   total
    Number of trades                13      19      32 
    Total trade volume (SOL)    0.1300 -0.1900 -0.0600
    Total trade volume (USDT)   -17.80   26.06    8.26
    Avg price                    136.9   137.1   137.0

  Assets:
                       start current  change 
    SOL               0.0997  0.0397 -0.0600
    USDT               16.09   24.35    8.26
    SOL-USDT price     138.4   136.3   -2.07
    Base asset %      46.19%  18.20% -27.98%

  Performance: 
    Hold portfolio value      29.69 USDT
    Current portfolio value   29.77 USDT
    Trade P&L                0.0774 USDT
    Fees paid                0.0208 USDT
    Fees paid                0.00010 SOL
    Total P&L                0.0423 USDT
    Return %                       0.14%
```

我在一边写文章，一边测试这个策略，正好赶上 SOL 的大波动，跑了 1 小时还赚了一点。

### 其他一些命令

- `stop`，停止这个机器人；
- `order_book`，查看当前的 `order_book`（右侧 Pane）,可加上 `--live` 实时监控；
- `ticker`，查看当前的价格，也支持 `--live` 选项；
- `import`，导入策略配置，看起来是用于导入老版的策略配置；

## 总结

我的感受，Hummingbot 相对其它的框架，会复杂一点，特别是开发一个新策略时。它也在不断迭代，基本每一两个月就有一个新版本发布。这篇文章只是简单演示了下它的使用。

Hummingbot 还有很多内置策略，除了演示的这个 `simple_pmm` 简单做市外，还有如跨交易所做市套利、资金费率套利、CEX/DEX 套利、网格、DCA等等。如果要做 DEX/CEX 的套利，配置的内容也挺多的。

这篇文章也没有介绍如何自己开发一个 Hummingbot 机器人，还有它的新版模块化架构，如何管理子策略。本来想多整理点的，但不想文章太长了。就到先到这里，后续慢慢来吧。

最后，希望本文对你有用！

