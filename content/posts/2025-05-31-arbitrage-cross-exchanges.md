---
title: "跨交易所套利的下单策略：速度、滑点与手续费的权衡"
date: 2025-05-31T10:00:00+08:00
draft: true
comment: true
description: "在所有套利交易中，下单策略必须在本质是成交速度、滑点控制和手续费成本三者间权衡。"
---

在所有套利交易中，下单策略的核心在于如何在成交速度、滑点控制与手续费成本三者之间取得平衡。成交速度越快，则滑点可能越大；而滑点越小，则可能需要挂单等待，影响成交；而吃单虽快，却需承担更高手续费。

本文将梳理一些可用于跨交易所套利的下单策略，阐述它们的优缺点

## 双边市价单成交

双边市价单成交方式最简单的套利下单策略。

它的优点是最快成交，可保证套利逻辑不会因为挂单没成交而破裂。而缺点也很明显，就是滑点最大，且手续费最高（通常都是 taker 吃单成交）。

## 双边超价限价单

双边超价限价单是跨交易所套利中常见的下单方式之一。

它通过在买卖两边分别设置略高于（买单）或略低于（卖单）当前盘口价格的限价单，实现近似市价单的效果。

速度方面，因为是超价成交，尽可能保留了市价成交的速度。而相对于市价下单，它设定一定的滑点容忍度，使得成交价格在可接受范围内。

其本质仍属于主动吃单，作为 taker 成交，手续费仍然相对较高。

超价订单可能出现被 "卡在盘口" 的情况，即限价单挂单却未被撮合。

可设置定期撤单重挂；
- 设定超时强制转为市价单；

等等方案以保障及时成交。









