---
title: "我的终端环境：iTerm2 的安装与体验"
date: "2023-09-28T19:23:22+08:00"
draft: false
comment: true
tags: ["zsh", "iterm2"]
description: "本系列的目标是介绍如何基于 iTerm2、zsh、Tmux 和 Neovim 搭建我的日常开发环境"
---

{{< video bb_id=619786097 yt_id=vNX_sReGZ5E >}}

本系列的目标是介绍如何基于 iTerm2、zsh、Tmux 和 Neovim 搭建我的日常开发环境。

本文是搭建我的开发环境系列中的第一篇，非常短小，将介绍如何安装与配置 iTerm2，安装成功后，即可阅读下一篇 [从 0 开始：教你如何配置 zsh](https://www.poloxue.com/posts/2023-09-16-how-to-use-zsh-a-beginner-guide/)。

## 前言介绍

首先，iTerm2 是一款终端软件，它是 macOS 下默认终端 Terminal 的替代品。我每次拿到电脑，或者因为某种原因重装了电脑系统后，首先要做的就是下载 iTerm2 以替换默认的终端 terminal。

iTerm2 相较于 Terminal 的优势功能，如分屏能力、主题配置、便利的搜索。另外，iTerm2 与 zsh 相结合体验更佳。

推荐阅读 [Terminal vs iTerm2: Comparing Two CLI Tools on macOS](https://techwiser.com/terminal-vs-iterm2-comparison/#:~:text=1.-,Multiple%20Panes,panes%2C%20in%20the%20same%20window.)

接下来，开始进入安装使用流程。

## 下载安装

安装可通过 iTerm2 [官网](https://iterm2.com/) 下载或者 MacOS 中 `brew` 安装，我将以 brew 安装为例。

如果还未安装 brew，安装命令：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装 iterm2，命令如下：

```
brew install --cask iterm2
```

等待执行完成，即安装完毕。

## 颜色面板

iTerm2 支持自定义颜色面板设置。我将以 Design Colours 为例介绍如何设置。

安装命令，如下所示：

```bash
curl -Ls https://raw.githubusercontent.com/MartinSeeler/iterm2-material-design/master/material-design-colors.itermcolors > /tmp/material-design-colors.itermcolors && open /tmp/material-design-colors.itermcolors
```

该命令的执行效果：

将会在 iTerm2 的 Preferences -> Color 下的 "Color Presets" 中新增一条颜色面板选项，即 "material-design-color"。选择该选项确认即可将 iTerm2 默认的颜色面板修改为 "material-design-color"。

想有更多的选择，再安装两个配色方案：


安装 Snazzy：

```bash
curl -Ls https://raw.githubusercontent.com/sindresorhus/iterm2-snazzy/main/Snazzy.itermcolors > /tmp/Snazzy.itermcolors && open /tmp/Snazzy.itermcolors
```

安装 Dracula:

```bash
curl -Ls https://raw.githubusercontent.com/dracula/iterm/master/Dracula.itermcolors > /tmp/Dracula.itermcolors && open /tmp/Dracula.itermcolors
```
 
查看 Color Presets 面板，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-install-iterm2-as-my-developing-environment-01.png)

选择一款你钟爱的配色，保存。

如果还没有满意的，可以从 [Iterm2-color-schemes](https://iterm2colorschemes.com/) 查找更多配色方案。使用类似如下的命令下载安装：

```bash
curl -Ls https://xxx > /tmp/{name}.itermcolors
open /tmp/{name}.itermcolors
```

## 使用体验

打开 'iTerm2'，快速使用体验一下吧。

### 分屏功能

如下图所示，"CommandL+d" 垂直分屏，"Command+D" 水平分屏。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-install-iterm2-as-my-developing-environment-02.jpeg)

### 分屏最大化

如下图所示，"Shift+Command+Enter" 分屏最大化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-install-iterm2-as-my-developing-environment-03.jpeg)

再次 "Shift+Command+Enter" 恢复分屏。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-install-iterm2-as-my-developing-environment-04.jpeg)

> 注：虽然是 iTerm2 的分屏是一个非常强大的能力，但我的工作环境的分屏主要还是基于 Tmux，它与 Vim 结合有更好的体验。

### 支持搜索

通过 "Command+f" 开启搜索框：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-install-iterm2-as-my-developing-environment-05.jpeg)

> 注：图片取自 [Terminal vs iTerm2: Comparing Two CLI Tools on macOS](https://techwiser.com/terminal-vs-iterm2-comparison/#:~:text=1.-,Multiple%20Panes,panes%2C%20in%20the%20same%20window)

### 插件集成

我的终端在通过 zsh + oh-my-zsh 配置后的效果图，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-install-iterm2-as-my-developing-environment-06.png)

## 最后

本文重点介绍了 iTerm2 的安装、颜色面板的配置，还有体验其核心的能力。

进一步学习 zsh 配置与插件安装，请阅读：[我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://www.poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)。这是我之前的一篇翻译文，我觉得挺符合这个系列的下一篇，没必要再写一篇。我的 zsh 主题使用的是 powerlevel10k。它的安装教程，会在单独一篇介绍我的常用插件时介绍。

