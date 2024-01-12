---
title: "我的终端环境：高效 shell 命令（一）之目录文件 exa、zoxide 与 bat"
date: 2023-10-27T15:01:19+08:00
draft: false
comment: true
description: "本篇是高效命令系列第一篇，介绍 exa、zoxide 、bat 替换 ls 和 cd、cat，为我们的终端增添色彩。"
tags: ["zsh"]
---

{{< video bb_id="235179220" yt_id="0yAP6ER8oGE" >}}

类 Unix 系统发展多年，不少老古董命令还在占据终端的绝大部分时间，而使用体验上却依然差强人意。

从本文开始，我将用一系列文章介绍提升终端效率的一系列命令，这些命令更具现代风格，希望能让你眼前一亮。

## 前言

正式开始前，先推荐一个 github 仓库 - [modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令，可用于替换一大波老古董命令，在这个系列的最后一篇，我将会整体过下该仓库中的所有命令。

本系列将要介绍的命令，我大概可将其分为三个大类，分别是：文件目录查看相关的命令 - [exa](https://github.com/ogham/exa)、[zoxide](https://github.com/ajeetdsouza/zoxide)、[bat](https://github.com/sharkdp/bat)，文件目录搜索相关的命令 - [fd](https://github.com/sharkdp/fd)、[ripgrep](https://github.com/BurntSushi/ripgrep)、[fzf](https://github.com/junegunn/fzf) ，HTTP Web 开发的相关命令 - [entr]()、[httpie](https://github.com/httpie/cli)、[jq](https://github.com/jqlang/jq)。

希望通过这些命令的学习，进一步提升我们的工作效率。

> 系列阅读：
>
> - [我的终端环境：iTerm2 的安装与体验](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [我的终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [我的终端环境：高效 shell 命令（一）之目录文件命令 exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [我的终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [我的终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [我的终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [我的终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [我的终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...


本文是高效命令系列第一篇，将先介绍平时工作中最常用的与目录文件相关的命令：

- [exa](https://github.com/ogham/exa)，替换默认的 ls 命令；
- [zoxide](https://github.com/ajeetdsouza/zoxide)，更智能地进行目录跳转，可替换默认的 cd 命令；
- [bat](https://github.com/sharkdp/bat)，与 cat 功能相似，但它支持语法高亮，实现了 git 的集成；

这三个命令直接替换掉最常用系统内置命令：ls、cd 与 cat。

> 注：exa 已停止维护，可用 [eza](https://github.com/eza-community/eza) 替代，它是基于 exa 的 fork 版本。安装命令：brew install eza。

## [exa](https://the.exa.website/)

这是一款可用于替换系统默认 ls 的命令，在平时工作中 ls 几乎使用最多的命令，而 exa 在支持 ls 的基本能力基础上，提供了更丰富的特性。

### 快速安装

```bash
brew install exa
```

### 使用

案例 1 - exa 默认配色效果；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-01.png)

案例 2 - exa \--icons 显示文件类型图标；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-02.png)

案例 3 - exa -alh \--git 或 exa \--all \--long \--header \--git，显示文件列表详情，和 git 信息；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-02.png)

案例 4 - exa \--tree \--icons，显示文件树；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-04.png)

### 别名

通过别名将默认 ls 和 tree 命令替换为 exa；

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

> 特别说明：别名生效后，如果还希望使用原始命令，可通过类似于 `\command` 的形式实现，如 `\ls` 将无效别名设置，直接使用系统内置 ls 命令。

## [zoxide](https://github.com/ajeetdsouza/zoxide)

提前问自己一个问题，Linux 默认命令 cd 好不好用？我的答案是，相当难用，无论多么丝滑的操作，一旦遇到 cd，只能说一句 fuck。

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


[zoxide](https://github.com/ajeetdsouza/zoxide) 是一款受到 z 和 autojump 启发而来的命令，它会记录访问过的目录，实现以 **最最最最** 少按键就能实现目录跳转。

> 注：前面的教程介绍了 oh-my-zsh 提供的 z 插件，zoxide 与 z 相比更易于使用。一份对比报告：[zoxide vs zsh-z](https://www.libhunt.com/compare-zsh-z-vs-zoxide)。

### 安装

```zsh
brew install zoxide
```

### 配置

在 zsh 中使用 zoxide，通过一行命令将 zoxide 初始化命令追加到 `~/.zshrc` 中，如下所示：

```zsh
## 通过 z 使用 zoxide
echo 'eval "$(zoxide init zsh --cmd z)"' >> ~/.zshrc
## 或直接替换 cd 命令
echo 'eval "$(zoxide init zsh --cmd cd)"' >> ~/.zshrc
```

我将会使用 zoxide 直接替换 cd 命令，即第二种配置方式。

### 使用

假设，有如下目录结构：

```
~/Hello
 |_ ./golang-examples
 |_ ./python-examples
 |_ ./rust-examples
 |_ ./trading-strategies
```

由于 zoxide 是通过历史访问路径的方式实现快速跳转的，要先准备数据。

如下命令初始化 zoxide 数据库：

```zsh
cd ~/Hello/golang-examples
cd ~/Hello/python-examples
cd ~/Hello/rust-examples
cd ~/Hello/trading-strategies
```

案例 1：cd golang-examples - 全名匹配；

案例 2：cd golang - 部分匹配；

案例 3：cd examples - 重名按算法选择最优目录；

案例 1、2、3 的演示效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-05.gif)

案例 4：cd examples + \<Space\>+\<Tab\>，进入交互选择模式；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-07.gif)

除了通过 \<Space\>+\<Tab\> 实现交互选择模式，还可直接使用 cdi 命令实现，直接看案例 5；

案例 5：cdi examples 直接进入交互选择模式；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-06.gif)

## [bat](https://github.com/sharkdp/bat)

[bat](https://github.com/sharkdp/bat) 是一款支持语法高亮与集成 GIT，功能与 cat 相似的命令。

### 安装

```zsh
brew install bat
```

### 配置

bat 的默认效果在我的 Macos 系统下的 tmux 模式下主题显示不够友好，每次要在选项上要配置主题，类似 `bat --theme=TwoDark main.py`。实际上，bat 命令提供了配置文件的能力。

> 可通过 bat --list-themes 查看 bat 支持的主题列表。

首先，在 `.zshrc` 中设置 bat 配置文件位置环境变量。

```
export BAT_CONFIG_PATH="${XDG_CONFIG_HOME:-~/.config}/bat.conf"
```

生效后，执行如下命令将会生成配置文件：

```zsh
bat --generate-config-file
```

配置 bat 默认选项，将主题配置 `--theme=TwoDark` 启用，如下所示：

```zsh
# Specify desired highlighting theme (e.g. "TwoDark"). Run `bat --list-themes`
# for a list of all available themes
--theme=TwoDark
```

### 使用

案例 1：语法高亮效果与行号；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-08.gif)

案例 2：集成 git 信息；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-09.gif)

案例 3：和 cat 一样，一次性无分页输出文本；

```zsh
bat --pager='never' logger.go
```

对于习惯使用 cat 的模式，如希望默认不启用分页能力，可直接在配置文件增加 `--paging=never`。

### 别名

觉得 bat 不错，想直接替换 cat 命令，在 `zshrc` 中配置别名即可，将默认 cat 命令，替换为 bat，如下所示：

```bash
alias cat='bat'
```

## 总结

本文介绍了三个命令，分别是 exa(eza)、zoxide 和 bat 的使用，让最常用最无聊的命令也能产生有趣的体验，实现使用效率的提升。

我的博文 - [我的终端环境：提升效率的 shell 命令（一）之目录文件命令 exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
