---
date: 2024-03-17
title: "自定义提示主题"
weight: 5
---

# 自定义提示主题

不知道你是否想过自定义 Shell 提示符主题能带来的不仅是终端美观度的提升，还能通过视觉优化增强工作效率呢？

在众多 shell 提示符主题中，Powerlevel10k 因为支持高度可定制和丰富的功能选项，非常值得推荐。本文基于这个主题介绍 zsh 主题插件 Powerlevel10k，包括它的安装和配置自定义。

## 安装 Powerlevel10k

首先安装 Powerlevel10k 本体：

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

然后在 `~/.zshrc` 中将主题设置为 Powerlevel10k：

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

执行 `source ~/.zshrc`，如果是第一次安装，p10k 配置向导会自动启动。

## 配置向导

Powerlevel10k 提供了一个交互式配置向导，你只需要回答几个问题即可完成配置：

```bash
p10k configure
```

向导会问你的偏好：

1. **主题风格** — 经典（雷击图标）还是极简（纯文字）
2. **文字布局** — 单行还是双行
3. **左侧信息** — 时间、路径、git 分支等
4. **右侧信息** — 时间、命令执行时长、shell 层级等
5. **连接线** — 圆角、直角、斜线还是无连接线
6. **字体** — 是否安装推荐字体（Nerd Font 或 Meslo Nerd Font）

每个问题都有实时预览，选完即生效。

## 信息区域

一个典型配置好的 Powerlevel10k 看起来长这样：

```
❯ ~/Projects/myapp on main ⇡1 ⇢2
❯ 
```

各段含义（从左到右）：

| 段 | 含义 |
|----|------|
| `❯` | 提示符符号 |
| `~/Projects/myapp` | 当前路径（缩短显示） |
| `on main` | 当前 git 分支 |
| `⇡1` | git 有 1 个提交未推送 |
| `⇢2` | git 有 2 个文件未暂存 |

右侧还显示上一个命令的执行时长和当前时间——命令跑太久了一目了然。

## 自定义配置

如果你不想走向导，也可以手动编辑 `~/.p10k.zsh`。这个文件每段配置都带了详细的注释说明。

常用的自定义：

```bash
# 修改左侧元素
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(dir vcs)

# 修改右侧元素
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status command_execution_time time)

# 显示完整路径（默认缩短家目录为 ~）
POWERLEVEL9K_DIR_PATH_ABSOLUTE=true

# 命令执行时间超过 3 秒才显示
POWERLEVEL9K_COMMAND_EXECUTION_TIME_THRESHOLD=3

# 修改图标
POWERLEVEL9K_VCS_GIT_ICON=''
```

## 字体要求

Powerlevel10k 推荐使用 Nerd Font 字体来显示图标。如果安装时没有安装推荐字体，可以手动安装：

```bash
# Meslo Nerd Font
brew install --cask font-meslo-lg-nerd-font
```

然后在 iTerm2 的 `Preferences > Profiles > Text > Font` 中选择 `MesloLGS Nerd Font`。

## 小结

Powerlevel10k 是目前 zsh 主题的天花板，速度快、配置灵活、信息丰富。安装加配置向导走一遍，你的终端提示符就从朴素的黑白文字，变成了信息密度极高的效率工具。
