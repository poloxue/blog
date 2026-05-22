---
date: 2024-02-18
title: "安装与主题"
weight: 1
---

# 安装与主题

本节介绍 oh-my-zsh 的安装与主题。

## 安装

oh-my-zsh 是 zsh 最流行的配置管理框架，安装极其简单：

```bash
# curl 安装
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 或者用 wget
sh -c "$(wget -O- https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装脚本会自动备份你当前的 `~/.zshrc` 为 `~/.zshrc.pre-oh-my-zsh`，然后创建一份新的 `~/.zshrc`。

## 目录结构

安装完成后，oh-my-zsh 的目录结构如下：

```
~/.oh-my-zsh/
├── custom/          # 自定义配置和插件
├── plugins/         # 内置插件（300+）
├── themes/          # 内置主题
├── lib/             # 核心函数库
├── templates/       # 模板文件
└── oh-my-zsh.sh     # 入口文件
```

## 切换主题

oh-my-zsh 内置了大量主题。修改 `~/.zshrc` 中的 `ZSH_THEME` 字段即可切换：

```bash
# ~/.zshrc
ZSH_THEME="robbyrussell"    # 默认主题
ZSH_THEME="agnoster"        # 经典主题，显示 git 信息
ZSH_THEME="ys"              # 简洁实用
ZSH_THEME="avit"            # 信息丰富的双行提示
ZSH_THEME="random"          # 每次随机换一个主题
```

设为 `random` 后，每次打开终端都是惊喜。如果遇到了特别喜欢的，zsh 会告诉你当前用的主题名。

## 常用主题推荐

- **agnoster** — 最受欢迎的第三方风格主题，显示 git 分支、状态等
- **ys** — 双行提示，上行列路径，下一行输入
- **candy** — 简洁的彩色双行主题
- **dst** — 左侧显示时间+路径，右侧显示 git 信息
- **bira** — 美观的箭头提示符

不过，更多高级用户会选择不依赖 oh-my-zsh 内置主题，而是安装 Powerlevel10k——它的自定义能力远超内置主题，下一篇会专门介绍。

## 小结

oh-my-zsh 的安装和主题切换就是这么简单。接下来我们看看它最强大的部分——插件系统。
