---
title: "无 oh-my-zsh 配置 zsh"
date: 2023-09-19T17:06:02+08:00
draft: true
comment: true
tags: ["shell", "zsh"]
---

大部分教程都在介绍如何用 oh-my-zsh 配置 zsh。但如果不用 oh-my-zsh （以下简称 OMZ）呢？是否可行？

本文将基于此展开介绍。

> 注：不得不说，脱离框架这种方式能让我们深入了解 zsh 的配置细节。无论什么工具，只有脱离框架，才能更好理解，也才能真正意义上用好框架。

事实是，OMZ 对于 zsh 并不是必选项，虽说，OMZ 是一款非常轻量级的框架，但它还是会让我的 Terminal 变慢。

不可接受！...

> 注：总得为这种瞎折腾找些理由。

还有什么其他方式吗？

其实，脱离了框架，依然能加载插件。通过类似如下的 script 即可实现：

```bash
source path/to/your/custom/plugin
```

我将一步步地介绍如何通过这种方式将原来以 OMZ 方式配置的主题或插件加载进来。

## 配置备份

开始前，将现在的 .zshrc 备份，如下命令：

```bash
cp ~/.zshrc ~/zshrc
```

## 主题配置

安装 spaceship 主题

译：[Using ZSH without OMZ](https://dev.to/hbenvenutti/using-zsh-without-omz-4gch)

