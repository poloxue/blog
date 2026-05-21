---
title: "exa"
weight: 1
---

# exa 

首先是 exa，一款可用于替换系统默认 ls 的命令，在平时工作中 ls 几乎使用最多的命令，而 exa 在支持 ls 的基本能力基础上，提供了更丰富的特性。

## 快速安装

```bash
brew install exa # 其他系统请查看 GithHub README.md
```

## 默认配色

首先，exa 默认提供了配色效果，无需 ls 要追加 `--color` 参数，省去了 alias 别名的设置。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-01.png)

## 显示图标

其次，exa 支持显示文件图片，通过指明 `--icons` 实现，显示文件类型图标；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-02.png)

## 详情与 GIT
更复杂的命令，支持 ls -l 显示文件列表详情，添加头部说明 header，如果是 git 仓库，可显示文件的 Git 信息。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-03.png)

## 目录树 Tree

如果想显示文件数，也不需要单独安装 tree 命令，如下 exa \--tree \--icons，显示文件树；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-04.png)

## 别名

如果你钟爱 exa 的这些能力，可通过别名将默认 ls 和 tree 命令替换为 exa。

如下是一些常见的别名设置：

```bash
# 默认显示 icons： 
alias ls="exa --icons"
# 显示文件目录详情
alias ll="exa --icons --long --header"
# 显示全部文件目录，包括隐藏文件
alias la="exa --icons --long --header --all"
# 显示详情的同时，附带 git 状态信息
alias lg="exa --icons --long --header --all --git"

# 替换 tree 命令
alias tree="exa --tree --icons"
```

如此一来，就将 exa 设置为系统默认 ls。

## 其他相似命令

除了 exa，还有两个可用于可替代 ls 的命令，分别名为 colorls 和 lsd，colorls 的性能太差，如果是目录中的文件数量多，能感觉到明显的卡顿。lsd 的号称是下一代的 ls，使用起来没有 colorls 那么卡，但性能上也不如 exa。

如果你对它们感兴趣，可以研究下。

## 注意

特别提醒，别名生效后，如果要用原始命令，要通过类似于 `\command` 的形式实现，如 `\ls` 将无效别名设置，直接使用系统内置 ls 命令。另外，如果你的 shell 脚本里使用了它，可能影响这些脚本的执行。这是设置别名替换系统命令要考虑的问题。
