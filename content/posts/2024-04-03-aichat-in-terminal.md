---
title: "2024 04 03 Aichat in Terminal"
date: 2024-04-03T18:21:36+08:00
draft: false
comment: true
description: "最近尝试了一款内置 AI 能力的终端软件，名为 Warp，它的交互设计非常不错，很值得上手。但它的问题是中文不友好，且我也不希望 AI 的能力被限制在某款终端上。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-03-aichat-in-terminal-01.png)

最近尝试了一款内置 AI 能力的终端软件，名为 Warp，它的交互设计非常不错，很值得上手。但它的主要问题是中文支持不够友好，且我也不希望 AI 的能力被限制在某款终端上。

在这个背景下，我开始尝试从网上搜索一些将 AI 引入终端的命令，发现了两款不错的软件，一个就是今天要介绍的这款名为 [tgpt](https://github.com/aandrew-me/tgpt) 的命令，无 API Key 要求，可以免费用。

本文带大家快速体验下这个命令。另特别提示，如遇网络问题，自行处理，

## 介绍

tgpt 是一个可以在终端上使用 ChatGPT，我觉得，它的最大卖点是无需 API Key 即可使用，毕竟免费才是王道。

**提到免费，ChatGPT 最近开放了无需注册即可使用，有条件的可以去体验下。**

tgpt 的默认模型是 phind，是一款对开发者友好的大模型。tgpt 支持多种模型，包括但不限于 KoboldAI、Phind、Llama2、Blackbox AI，以及需要 API 密钥的 OpenAI 所有模型和 Craiyon V3 图像生成模型。


接下来，开始快速安装体验下吧。

### 安装

```shell
curl -sSL https://raw.githubusercontent.com/aandrew-me/tgpt/main/install | bash -s /usr/local/bin
```

### 使用

它的使用方法也非常简单，只要在终端中输入如下命令：

```shell
tgpt "What is internet?"
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-03-aichat-in-terminal-02.gif)

或者，你也可以使用指定的模型：

```shell
tgpt --provider opengpts "What is 1+1"
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-03-aichat-in-terminal-03.gif)

## 场景

tgpt 另一个不错的点是在于它默认提供了三个场景的支持，分别是生成 shell 命令、代码和图片。

我们来具体测试下。

**生成 shell 命令**

Warp AI 的一大亮点就是自动生成命令的能力

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-03-aichat-in-terminal-04.gif)

生成命令后，提示我们是否直接执行。这肯定是终端场景下的必备能力。

**生成 Code**

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-03-aichat-in-terminal-05.gif)

最后，生成图片的效果就不展示了，效果上不错好，估计是提示词的要求比较高，要看下如何使用它的图片生成模型 Craiyon V3。

## 对话

tgpt 还支持对话模式，可以使用 tgpt -i 进入对话模式。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-04/2024-04-03-aichat-in-terminal-06.gif)

这样一来，对于大部分场景下，我们都不用开启浏览器了，省心啊。

## 最后

我觉得 tgpt 这个命令的最大优势在于它可以免费使用，而且提供了对程序员友好的一些模型和使用场景。初步看了下，它好像是没有提供自定义 role 的能力，这算是它的一个缺点吧。


