---
title: "我的终端环境：无用小知识 - neofetch/fastfetch 定制终端启动消息"
date: 2023-11-14T17:56:34+08:00
draft: true
comment: true
description: "本文接着上文，介绍终端启动消息的配置，让你的终端比之前文更加绚丽多彩。"
---

本文接着上文，介绍终端启动消息的配置，让你的终端比之前文更加绚丽多彩。

## 前言

你是否看到类似如下这样的终端启动消息？

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-01.png" >}}

我在刚开始讲终端环境这个系列，就有小伙伴在我的视频下 show 了他的终端。

{{< image "./2023-11-16-beautify-your-terminal-welcome-using-fetch-02.webp" >}}

今天就来讲讲这个终端效果是如何实现的吧。

## fetch

什么是 fetch?

neofetch

- 安装
```
brew install neofetch
```
- 使用
```
> netfetch
```
- 配置

```
brew install fastfetch
```
- iterm2 背景图设置

https $(https ://api.github.com/repos/poloxue/images/branches/main | jq -r '.commit.commit.tree.url') | jq '.tree[].path'
