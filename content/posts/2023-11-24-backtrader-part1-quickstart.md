---
title: "backtrader Part1 - 介绍、安装与快速体验"
date: 2023-11-21T19:33:23+08:00
draft: true
comment: true
description: "本文是 backtrader 的起始篇，介绍如何使用 backtrader 回测我们的量化策略。"
---

本文是 backtrader 的起始篇，介绍如何使用 backtrader 回测我们的量化策略。

## 什么是策略回测？

想象一个场景，某天你突然感觉有一个交易策略能快速致富，接下来你该怎么办呢？

普通散户因没有一套快速验证策略有效性的方法论，这时有两种选择，要不是理科用这种策略实盘交易，亦或谨慎些的朋友，用模拟账号先交易一段时间测试效果。

但毫无疑问，两者皆是不好的选择，无论是看似合理的通过模拟账号交易测试一段时间，还是看起来就非常糟糕的第一种方式。

## 什么是 backtrader？

backtrader 是一款集策略回测、评估分析与实盘交易的强大 Python 策略回测框架。前面提到，可利用历史数据验证策略的有效性，backtrader 正是这样一款支持该能力的工具。

- 什么是 backtrader
  - 单品种 CTA 策略；
  - 多品种套利/组合策略；
  - 技术方案：
    - 事件驱动；
      - 基于时间；
      - 基于消息；
    - 矢量回测
- python 环境：
  - anconda；
  - brew install python；
  - virtualenv 虚拟环境；
- 环境版本；
  - virtualenv
  - python 3.9
  - pip install backtrader
  - requirements.txt；

