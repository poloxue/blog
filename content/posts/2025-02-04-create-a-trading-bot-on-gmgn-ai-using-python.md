---
title: "基于 Python 开发 GMGN.AI 抢币机器人"
date: 2025-02-04T11:51:45+08:00
draft: false
comment: true
description: "本文介绍如何使用 Python 开发一个 gmgn.ai 的抢币机器人，它可以用来从链上抢币，如 Trump 等 meme 币。"
---

本文介绍如何使用 Python 开发一个 gmgn.ai 的抢币机器人，它可以用来从链上抢币，如 Trump 等 meme 币。

**风险提示：** GMGN.AI 的币波动极大，有大量的空气币，务必谨慎操作，切勿在不了解的情况下投入过多资金。本文是我学习过程中的记录，请仔细甄别，防止资金损失。

完整的实例代码请查看：[sniper_new.py](https://gist.github.com/poloxue/0777e9808747981ff935aaaa1140a73b)。

## 什么是 GMGN.AI?

GMGN.AI 是一个代币追踪和分析平台，提供了实时市场信号和自动交易机器人，我们可以监听它提供的实时信号，跟单聪明钱包，快速抓住赚钱机会。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-02/2025-02-04-create-trading-bot-on-gmgn-ai-using-python-01.png)

通过 GMGN.AI 的分析能力筛选潜力代币，然后在它上就可以直接交易。

除此以外，它还提供了自动化交易机器人。GMGN.AI 的自动化交易机器人都是依托于 telegram 实现的，它提供了很多机器人，如新币信号、聪明钱包等，都可以通过监听 telegram 消息实现，然后在基于它提供的交易机器人直接下单交易。

GMGN.AI 内置机器人功能是固定的，如果我想自定义交易规则，如按持有人数、流动市值等过滤掉一些不满足条件的币，就需要自己写程序实现了。

## 如何实现一个抢币机器人

在 GMGN.AI 实现自定义交易机器人高度依赖 Telegram，要通过 telegram 的 app 将我的交易机器人与 GMGN.AI 的 telegram 机器人连接。我将使用  **Python** 实现这个机器人。

实现的大致流程如下：

1. **创建 Telegram App**：在 Telegram 上创建一个应用，获取 `api_id` 和 `api_hash`。
2. **监听 GMGN.AI 信号**：GMGN.AI 会实时推送不同类型信号给 Telegram，机器人要监听刚兴趣的信号进行抢单。
3. **按条件过滤信号**：监听到信号消息，将消息文本转为结构化的数据，判断是否下达交易。
4. **通过 Sniper Bot 执行交易**：一旦确认某个币符合条件，发送指定给 Sniper Bot 下单。

## 安装依赖包

提前安装好依赖库：

```bash
pip install telethon instructor pydantic openai
```

`Telethon` 库是用于和 telegram 交互的，而 `instructor` 是要与大模型配合将 Telegram 文本消息结构化的；

## 创建 Telegram App

访问 [Telegram Developer](https://my.telegram.org/apps)，使用 Telegram 账户登录。接下来会看到如下的页面。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2025-02/2025-02-04-create-trading-bot-on-gmgn-ai-using-python-02.png)

填写这个表单，创建应用，就能获得认证所要的 `api_id` 和 `api_hash`，实现程序和 Telegram 交互的身份验证。

## 验证登录

正式开始机器人的逻辑前，先用 `Telethon` 来登录 Telegram，验证下 `api_id` 和 `api_hash` 是否正确。

实现代码如下：

```python
from telethon import TelegramClient, events

api_id = 'YOUR_API_ID'
api_hash = 'YOUR_API_HASH'

# 创建 Telegram 客户端
client = TelegramClient('session_name', api_id, api_hash)

async def main():
    # 登录 Telegram
    await client.start()
    print("登录成功！")

    # 获取当前用户信息
    me = await client.get_me()
    print(f"用户名：{me.username}，ID：{me.id}")

    await client.run_until_disconnected()

client.loop.run_until_complete(main())
```

如果认证成功，会打印 "登录成功" 和我的用户信息。

```bash
登录成功！
用户名：xxx，ID：xxx
```

## 信号来源

GMGN.AI 提供一些信号群，如 t.me/gmgnsignals 和 t.me/gmgnsignalsol， 会实时推送不同主题的信号消息。

如下是 GMGN.AI 的官方文档上给出的信号列表：

- CTO信号 (社区接管）
- Update DEX Screener Social Infos (更新社交三件套)
- PUMP Update DEX Screener Social Infos (PUMP币更新社交三件套)
- PUMP FDV Surge (PUMP内盘代币快速飙升)
- Solana FDV Surge (Solana代币快速飙升)
- Smart Money FOMO (聪明钱FOMO)
- KOL FOMO
- DEV Burnt Alert (作者烧币)
- ATH Price (历史新高)
- Heavy Bought (大单买入)
- Sniper New (狙新币)

我将以监听 "Sniper New（狙击新币）" 消息为例，演示如何狙击满足我要求的新币。

## 监听 "Sniper New" 消息

首先，"Sniper New" 消息位于 https://t.me/gmgnsignalsol 群组下。我将用 `telethon` 的消息监听能力监听 `@gmgnsignalsol` 下的 "Sniper New" 消息。


先查看下消息的格式：

```bash
🎯Featured New Pair🎯

$PEPSI(Pepsi On Solana)
4ZsJoreG6w6xpgHPtaBBA6aSr8EzHVoDdezAkxeFVJvG

📈 5m | 1h | 6h: >99999% | >99999% | >99999%
🎲 5m TXs/Vol: 54/$93.9K
💡 MCP: $857.1M
💧 Liq: 453.11 SOL ($186.8K 🔥100%)
👥 Holder: 136
🕒 Open: 16s ago

✅ NoMint / ✅Blacklist / ✅Burnt
✅TOP 10: 13.52%

⏳ DEV: Add Liquidity
👨‍🍳 DEV Burnt烧币: 0(🔥Rate: %)

Backup BOT: US | 01 | 02 | 03 | 04

🌏 Website | ✈️ Telegram

🌈 NEW: Wallet Copy Trading is Live! Click NOW
```

这个消息的特点是开头包含 "Featured New Pair" 文本。

如下是实现代码：

```python
@client.on(events.NewMessage(
    chats="@gmgnsignalsol", pattern=".*Featured New Pair.*"
))
def on_new_sniper(event):
    print(event.message.text)
```

如上的代码，`events.NewMessage` 的两个参数 `chats` 和  `pattern` 是用来限定监听什么样的信号。

到此，我的这个机器人就能实时监听新币消息了。

## 文本消息结构化

现在这个消息还是是文本内容，并不便于接下来的分析，程序中起码要能拿到如代币的合约地址、持有人数、市值的等信息。最起码的，要能拿到合约地址吧。

如何将文本结构化呢？

一种常用方式是通过正则解析，但这要求文本格式固定，稍微有所改变就要重新解析。

还有一种方式，通过 LLM 大语言模型实现文本结构化。deepseek 的出现让 LLM 的调用成本非常的低。如果在意成本，跑一个本地模型也能满足需求。

首先要让大模型知道要提取什么数据，通过 `pydantic` 将目标定义出来。

代码如下所示：

```python
from pydantic import BaseModel, Field
from datetime import datetime

class NewPair(BaseModel):
    contract_address: str = Field(..., description="The contract address, used to uniquely identify the contract")
    market_cap: float = Field(..., description="The market capitalization of the contract, usually in USD")
    liquidity_sol: float = Field(..., description="Liquidity in SOL")
    liquidity: float = Field(..., description="Liquidity in USD")
    holders_count: int = Field(..., description="The number of holders of the contract")
    open_seconds: int = Field(..., description="The number of seconds since the contract was opened")
    top10_ratio: float = Field(..., description="The ratio of the top 10 holders, expressed as a decimal")
    no_mint: bool = Field(..., description="Whether the contract is NoMint, meaning no new tokens can be minted")
    blacklist: bool = Field(..., description="Whether the contract is blacklisted")
    burnt: bool = Field(..., description="Whether the tokens in the contract are burnt")
```

接下来，用 LLM 按定义的模型从文本抽取数据即可。我用到一个名为 `instructor` 简化这个过程。

实现代码如下：

```python
import instructor
from openai import AsyncOpenAI

llm = instructor.from_openai(AsyncOpenAI(
    base_url="https://api.deepseek.com",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
))

@client.on(events.NewMessage(
    chats="@gmgnsignalsol", pattern=".*Featured New Pair.*")
)
async def on_sniper_new(event):
    response = await llm.chat.completions.create(
        model="deepseek-chat",
        response_model=NewPair,
        messages=[{"role": "user", "content": event.message.text}],
    )
    print(response)
```

输出样例如下：

```bash
contract_address='EX82HnkDihYGP8owXSDZfyMQXzovvYiWddNAxuxfpump' market_cap=87400.0 liquidity_sol=81.94 liquidity=34400.0 holders_count=523 open_seconds=120 top10_ratio=0.2207 no_mint=True blacklist=True burnt=True
```

现在，要按选币条件过滤新币，如 top10 的占比要小于 10%，只有人数要大于 500 人，流动市值要大于 100k USD。

```python
async def on_sniper_new(event):
    r = await llm.chat.completions.create(
        model="deepseek-chat",
        response_model=NewPair,
        messages=[{"role": "user", "content": event.message.text}],
    )
    if (
        r.holders_count > 500
        and r.liquidity > 10e5
        and r.top10_ratio < 0.1
        and r.blacklist
        and r.no_mint
        and r.burnt
    ):
        await create_buy_order(r.contract_address, 0.01) # 下单 0.01 SOL

```

到此，我已经按规则选择出来了我想要的币。现在还差最后一步，实现下单交函数 `create_buy_order`。

## 通过 Sniper Bot 执行交易

GMGN.AI 提供了在 Telegram 通过发送命令执行交易的机器人，如实现自动化交易。 SOL 连上的抢币机器人，请查看官方文档：[TG Solana 狙击机器人](https://docs.gmgn.ai/cn/tg-solana-ju-ji-ji-qi-ren)。

快速学习几个简单的使用机器人的命令案吧。

- 立刻买入某合约 0.1 SOL：
```bash
/buy EX82HnkDihYGP8owXSDZfyMQXzovvYiWddNAxuxfpump 0.1
```

- 立刻卖出持有仓位的 50%：

```bash
sell EX82HnkDihYGP8owXSDZfyMQXzovvYiWddNAxuxfpump 50
```

还提供了很多其他的命令，如仓位、钱包余额、当前订单等等。

到此，`create_buy_order` 实现起来就很简单了，如下所示：

```python
async def create_buy_order(contract_address, amount):
    buy_command = f"/buy {contract_address} {amount}"
    await client.send_message("@GMGN_sol03_bot", buy_command)
    print(f'已执行交易: {buy_command}')
```

到此，整个流程就打通了，接下来的重点就是找到能赚钱的策略了。

## 总结

本文介绍了如何利用 Python 的 **Telethon** 库结合 **GMGN.AI** 提供的信号和 **Sniper Bot** 执行自动化抢币交易。

再次说明：我开发这个机器人仅仅为学习目的，去打通了抢币的自动化流程。链上交易风险很大，要想真正赚钱，还要自行下功夫研究。
