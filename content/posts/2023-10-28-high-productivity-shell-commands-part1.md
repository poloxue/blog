---
title: "用 exa/zoxide/bat 替换 ls/cd/cat 命令"
date: 2023-10-27T15:01:19+08:00
draft: false
comment: true
description: "本篇是高效命令系列第一篇，介绍 exa、zoxide 、bat 替换 ls 和 cd、cat，为我们的终端增添色彩。"
tags: ["zsh"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-00.png)

如下是视频版本，没有文章详细。

{{< video bb_id="235179220" yt_id="0yAP6ER8oGE" >}}

类 Unix 系统发展多年，不少古董命令还在占据终端的绝大部分时间，但它们的使用体验上却是差强人意。从本文开始，我计划用几篇文章介绍提升终端效率的一系列命令，它们更具现代风格，希望能让你眼前一亮。

## 前言

本文是高效命令系列第一篇，将先介绍平时工作中最常用的与目录文件相关的命令，分别是替换 ls 的 [exa](https://github.com/ogham/exa)，替换 cd 的 [zoxide](https://github.com/ajeetdsouza/zoxide) 和替换 cat 的 [bat](https://github.com/sharkdp/bat)。它们的优势会在文章中逐步展开说明。

> 特别提醒：exa 已停止维护，可用 exa 的 fork 版本 [eza](https://github.com/eza-community/eza) 替代。

正式开始前，先推荐一个 github 仓库 - [modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令，可用于替换一大波老古董命令，在这个系列的最后一篇，我将会整体过下该仓库中的所有命令。希望通过这些命令的学习，能进一步提升我们的效率。

## [exa](https://the.exa.website/)

首先是 exa，它是一款可用于替换系统默认 ls 的命令，在平时工作中 ls 几乎使用最多的命令，而 exa 在支持 ls 的基本能力基础上，提供了更丰富的特性。

### 快速安装

```bash
brew install exa # 其他系统请查看 GithHub README.md
```

### 使用

首先，exa 默认提供了配色效果，无需 ls 要追加 `--color` 参数，省去了 alias 别名的设置。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-01.png)

其次，exa 支持显示文件图片，通过指明 `--icons` 实现，显示文件类型图标；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-02.png)

更复杂的命令，支持 ls -l 显示文件列表详情，添加头部说明 header，如果是 git 仓库，可显示文件的 Git 信息。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-03.png)

如果想显示文件数，也不需要单独安装 tree 命令，如下 exa \--tree \--icons，显示文件树；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-04.png)

### 别名

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

除了 exa，还有两个可用于可替代 ls 的命令，分别名为 colorls 和 lsd，colorls 的性能太差，如果是目录中的文件数量多，能感觉到明显的卡顿。lsd 的号称是下一代的 ls，使用起来没有 colorls 那么卡，但性能上也不如 exa。

如果你对它们感兴趣，可以研究下。

特别提醒，别名生效后，如果要用原始命令，要通过类似于 `\command` 的形式实现，如 `\ls` 将无效别名设置，直接使用系统内置 ls 命令。另外，如果你的 shell 脚本里使用了它，可能影响这些脚本的执行。这是设置别名替换系统命令要考虑的问题。

我们继续介绍下一个命令 - zoxide。

## [zoxide](https://github.com/ajeetdsouza/zoxide)

在正式介绍 zoxide 前，尝试提前问自己一个问题，Linux 默认命令 cd 好不好用？我的答案是，相当难用，无论多么丝滑的操作，一旦遇到 cd，只能说一句 f**k。

你是不是经常这样使用 cd 呢？

```
cd ../
cd ../
cd ../
cd ../../../
cd x/
cd y/
cd z/
```

如果不想被 cd 折磨的话，我强烈推荐这个工具：[zoxide](https://github.com/ajeetdsouza/zoxide)。

[zoxide](https://github.com/ajeetdsouza/zoxide) 是一款受到 z 和 autojump 启发而来的命令，它会记录访问过的目录，通过搜索找到最匹配你目标的目标。从而实现以 **最最最最** 少按键就能实现目录跳转。

一般情况下，我们关注的目录就那几个，90% 的情况用它的快速跳转能力即可，而一些特殊情况，cd 绝对路径即可，亦或者是使用它提供的另一种方式，交互式搜索。

前面我写过一篇文章 介绍了 oh-my-zsh 提供的 z 插件，zoxide 与 z 相比更易于使用。这有一份对比报告：[zoxide vs zsh-z](https://www.libhunt.com/compare-zsh-z-vs-zoxide)。

具体介绍它的安装使用吧。

### 安装

```zsh
brew install zoxide # 其他系统请查看 GithHub README.md
```

### 配置

在 zsh 中使用 zoxide 要简单配置下，一行命令将 zoxide 初始化命令追加到 `~/.zshrc` 中。

如下所示：

```bash
echo 'eval "$(zoxide init zsh --cmd z)"' >> ~/.zshrc
```

生效后，即可通过 z 命令使用 zoxide 的能力。

亦或者，如果觉得要从习惯于使用 cd 切换到 z 有难度，可直接将 z 命名为 cd，直接替换掉系统的 cd 命令，配置 `--cmd cd` 即可。
```bash
echo 'eval "$(zoxide init zsh --cmd cd)"' >> ~/.zshrc
```

我将会使用 zoxide 直接替换 cd 命令进行演示，即第二种配置方式。

### 使用

正式开始使用前，先假设我有如下目录结构：

```
~/Hello
 |_ ./golang-examples
 |_ ./python-examples
 |_ ./rust-examples
 |_ ./trading-strategies
```

由于 zoxide 是在历史访问路径的基础上智能选择。为了便于演示，我把之前的访问历史记录已经清空了，并通过如下命令初始化访问历史：

```zsh
cd ~/Hello/golang-examples
cd ~/Hello/python-examples
cd ~/Hello/rust-examples
cd ~/Hello/trading-strategies
```

假设现在在 `~` 用户目录下，如下是演示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-05.gif)

主要演示了 zoxide 的三个能力，分别是全名匹配、部分匹配和如果存在重名目录下会选择智能最优的目录。

首先是全名匹配，通过 zoxide 的 cd 命令，在 `~` 目录下快速跳转到 golang-examples 目录，直接输入 `cd golang-examples`。

其次是部分匹配，由于记录中只有一个目录包含 python，执行 cd python 会直接进入到 `python-examples`；

最后是重名按算法选择最优目录，我不清楚它的具体算法，但观察来看，它当前在 python-examples 目录下，执行 cd examples 会优先找到最近访问的包含 examples 的目录，且非当前位置。

如果觉得它的智能算法不够灵活，还可以尽量补全路径，和使用普通 cd 一样。如果还想要它的效率，也可以进入它的交互搜索模式，zoxide 支持两种方式进入交互模式：

一种是输入 cd + 目标名称 + 快捷键 \<Space\>+\<Tab\>，进入交互选择模式，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-07.gif)

另一种是直接使用 cdi 命令也可进入交互搜索模式，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-06.gif)

补充，zoxide 本身也是个命令，你可以用它增删改查，管理历史方案记录。

```bash
$ zoxide query golang # 返回最匹配 golang 的目录
$ zoxide query golang --list # 返回所有匹配 golang 的目录
$ zoxide -h # 更多帮助信息
zoxide 0.9.2
Ajeet D'Souza <98ajeet@gmail.com>
https://github.com/ajeetdsouza/zoxide
一个更智能的终端 `cd` 命令

使用方法：
  zoxide <命令>

命令：
  add     添加一个新目录或增加其排名
  edit    编辑数据库
  import  从另一个应用导入条目
  init    生成 shell 配置
  query   在数据库中搜索目录
  remove  从数据库中移除目录
```

差不多，zoxide 大概就介绍的这么多吧。希望对于想用它替换掉 cd 的朋友有所帮助吧。

## [bat](https://github.com/sharkdp/bat)

说完了 ls 列举目录，cd 进入目录，我们继续介绍一个命令，[bat](https://github.com/sharkdp/bat) 查看文件内容。

首先，bat 和 Baidu/Alibaba/Tencent 没有联系，它是一款支持语法高亮、GIT 集成的用于替换类 Unix 系统下快速查看文件内容的命令，功能与 cat 相似的命令。

我们直接介绍它的安装与使用吧。

### 安装

```zsh
brew install bat # 其他系统请查看 GithHub README.md
```

### 使用

对于 bat 命令，我先介绍它的使用，然后再谈配置，因为配置并非它的必选项而是优化项。

bat 相比于 cat 的第一个优势，就是它支持语法高亮效果与行号显示。如我们查看一个 Go 的源码文件，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-08.gif)

而且，bat 还集成 Git。如下我们修改了 logger.go 文件，通过 bat 即可查看它的修改点；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-09.gif)

默认情况下，bat 采用分页输出，这对于读取大文件非常有帮助，不用担心失误导致产生一大片控制台输出。但如果你希望 bat 和 cat 一样，一次性无分页输出文本，可通过 `--pager=never` 或 `--no-pager` 选项实现。

```zsh
bat --pager=never logger.go
bat --no-pager logger.go
```

如果你习惯使用 cat 的模式，希望默认不启用分页能力，可直接在配置文件配置默认行为，在其中增加 `--pager=never`。

接下来说说如何通过 bat 的配置改变它的默认行为吧。

### 配置


bat 的配置文件路径是通过环境变量指定的。我们在 `.zshrc` 中设置 bat 配置文件位置环境变量。

```bash
export BAT_CONFIG_PATH="${XDG_CONFIG_HOME:-~/.config/bat.conf"
```

生效后，执行如下命令将会生成配置文件：

```zsh
bat --generate-config-file
```

生成配置文件，位于 `~/.config/bat.conf`。

假设我不喜欢 bat 默认的主题，就可以通过配置修改了。如配置 bat 默认选项，将主题改为 `--theme=TwoDark` 启用：


```zsh
# Specify desired highlighting theme (e.g. "TwoDark"). Run `bat --list-themes`
# for a list of all available themes
--theme=TwoDark
```

如果你想查看更多主题，可通过 `bat --list-themes` 查看 bat 支持的主题列表。

现在，不想启用 bat 的分页能力，在配置中添加：

```bash
--pager=never
```

### 别名

觉得 bat 不错，想直接替换 cat 命令，在 `zshrc` 中配置别名即可，将默认 cat 命令，替换为 bat，如下所示：

```bash
alias cat='bat'
```

## 总结

本文介绍了三个命令，分别是 exa(eza)、zoxide 和 bat 的使用，都是日常工作中最常用的三个命令。利用好它们，最常用最无聊的命令也能产生有趣的体验，实现效率提升。

最后，希望你喜欢这篇文章，且真正给你带来了帮助。

