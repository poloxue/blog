---
title: "安装与主题"
weight: 2
---

# 安装与主题

本节介绍 iTerm2 安装与主题。

## 安装

首先是安装，可通过 [iTerm2 官网](https://iterm2.com/) 下载或者 MacOS 中 `brew` 安装，我将以 brew 安装为例。

如果还未安装 brew，安装命令：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装 iterm2，命令如下：

```bash
brew install --cask iterm2
```

等待执行完成，即安装完毕。

## 主题

iTerm2 支持自定义主题，即设置颜色面板 color preset。我将先以 material design colors 为例，介绍如何安装设置。

它的安装命令，如下所示：

```bash
curl -Ls https://raw.githubusercontent.com/MartinSeeler/iterm2-material-design/master/material-design-colors.itermcolors > /tmp/material-design-colors.itermcolors && open /tmp/material-design-colors.itermcolors
```

如果成功执行，将会在 iTerm2 的 Preferences （使用快捷键 CMD+, 可快捷打开）-> Color 下的 "Color Presets" 中新增一条颜色面板选项，即 "material-design-color"。选中确认即可将 iTerm2 默认的颜色面板修改为 "material-design-color"。

上面的命令分为两步：首先通过 `curl` 下载配色文件，然后再通过 `open` 打开下载即可直接安装。

稍微麻烦一点做法是，可通过设置面板 import 导入下载的后缀为 itermcolors 的文件。

如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-07-v1.png)

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

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-01.png)

选择一款你钟爱的配色，保存。

如果还没有满意的，可以从 [Iterm2-color-schemes](https://iterm2colorschemes.com/) 查找更多配色方案

我从中选择了主题进行安装。如下是主题切换的效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-18.gif)



