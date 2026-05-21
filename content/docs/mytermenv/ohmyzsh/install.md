---
title: "安装与主题"
weight: 1
---

# 安装与主题

本节介绍 ohmyzsh 的安装与主题。

## 安装

oh-my-zsh 的安装很简单。安装命令，如下所示：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装后，就已经有一些默认效果，如命令行提示符的主题变化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-01.png)

这是默认的  oh-my-zsh 主题 "robbyrussell"。

## 主题

oh-my-zsh 提供了许多内置主题，可查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) 获取一系列的主题。

我们可直接通过 ~/.zshrc 配置更新主题配置，将内容修改如下：

```bash
ZSH_THEME="agnoster"` # 默认为 robbyrussell
```

执行 `source ~/.zshrc` 生效配置，就能看到主题效果。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-02.png)

另外，oh-my-zsh 还提供了 random 主题，它会在 oh-my-zsh 内置主题中随机选择一个主题展示。只需编辑 `~/.zshrc`，将 ZSH_THEME 更新为 random 即可。

配置如下所示：

```bash
ZSH_THEME="random"
```

演示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-03.gif)

说实话，我觉得没人会这么用吧。这明显很鸡肋的功能啊。

