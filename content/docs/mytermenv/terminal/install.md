---
date: 2024-01-14
title: "安装与主题"
weight: 2
---

# 安装与主题

本节介绍 iTerm2 安装与主题。

## 安装

安装 iTerm2 很简单，直接去官网 [iterm2.com](https://iterm2.com) 下载安装包，或者用 Homebrew：

```bash
brew install --cask iterm2
```

安装完成后，打开 iTerm2，你会看到默认的黑底白字界面。别急，我们先给它换个好看的主题。

## 主题配置

iTerm2 支持丰富的主题配色，社区维护了大量配色方案。推荐几个经典主题：

### 内置主题

iTerm2 内置了 100+ 配色方案，通过 `Preferences > Profiles > Colors > Color Presets` 即可切换。

### 安装社区主题

从 [iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes) 下载你喜欢的主题，然后在 Color Presets 中选择 Import 导入。

### 我的推荐

- **Dracula** — 深色主题经典之选，对比度舒适
- **Solarized Dark** — 护眼低对比度，适合长时间工作
- **One Dark** — Atom 编辑器同款，代码高亮清晰
- **Nord** — 冷色调，科技感十足

## 字体设置

好马配好鞍，一个好字体能大幅提升终端体验。推荐 Nerd Font 系列的等宽字体，它们额外集成了大量图标字符，配合 powerlevel10k 等工具显示效果极佳。

安装推荐字体：

```bash
# FiraCode Nerd Font
brew install --cask font-firacode-nerd-font

# JetBrains Mono Nerd Font
brew install --cask font-jetbrains-mono-nerd-font
```

然后在 `Preferences > Profiles > Text > Font` 中选择安装的字体，记得开启 Anti-aliased 和 Use a different font for non-ASCII text。

## 小结

安装和主题配置完成后，你的 iTerm2 已经告别了默认 Terminal 的朴素外观。下一篇我们开始实际使用，看看 iTerm2 的日常操作技巧。
