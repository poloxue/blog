---
title: '我的终端环境：与众不同的 zsh 主题 - powerlevel10k'
date: "2023-10-20T10:25:36+08:00"
draft: false
comment: true
description: "本教程介绍如何安装 zsh 主题 powerlevel10k 的安装与配置。"
tags: ["zsh"]
---

{{< video bb_id=620048257 yt_id=3FmaWzqtWZ0 >}}

本文介绍 zsh 主题 powerlevel10k 的安装与配置。

## 什么是 powerlevel10k?

Powerlevel10 是一款 zsh 的主题，它强调性能、灵活性和开箱即用。之前，我已经介绍了一些 zsh 主题，而通过 p10k（powerlevel10k 的简称）的可配置化能力，同样能配置出覆盖出之前主题的类似效果。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-13.png" >}}

> 系列阅读：
>
> - [我的终端环境：iTerm2 的安装与体验](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [我的终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [我的终端环境：高效 shell 命令（一）之目录文件命令 - exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [我的终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [我的终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [我的终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [我的终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [我的终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...

## NerdFont 字体安装

安装 powerlevel10k 前，先安装它依赖的字体：[NerdFont](https://github.com/ryanoasis/nerd-fonts#font-installation)。不同系统的安装方法，可查看[它的文档](https://github.com/ryanoasis/nerd-fonts#font-installation)。

MacOS 的话，可直接通过 Homebrew 快速安装：

```bash
brew tap homebrew/cask-fonts
brew install font-hack-nerd-font
```

安装完成，配置终端字体，进入 iTerm2 Settings -> Profiles -> Text -> Font -> MesloLGS NF 即可。

## 安装 powerlevel10k

通过如下命令 下载：

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

配置 `~/.zshrc`，启用 `powerlevel10k` 主题：

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

重新终端，或执行 `source ~/.zshrc` 生效配置。

## 配置 powerlevel10k

执行 `source ~/.zshrc` 后，终端将进入到 powerlevel10k 的配置向导。

### 前置问题

为了确认字体安装是否成功，在正式开始配置前，要先回答几个问题才能进入到主题的自定义向导。

问题有诸如：

- Does this look like a diamond (rotated square)? 这是看起来钻石吗？
- Does this look like a lock? 这看起来是锁吗？
- Does this look like an upwards arrow? 这看起来是向上箭头吗？

等等。

如果字体已经配置成功，这些问题看起来就像逗傻子一样，按实际情况回答问题即可。一般情况下，都是选择 "Yes"。

### 开始配置

进入到正式阶段，按步骤配置自己的提示符风格，下面一共 12 步，都是非常简单的回答问题。考虑到第一次配置，有懵逼的可能性，我把全部的步骤都列出来了。

1. 选择提示符风格，分别是 Lean、Classic、Rainbow 和 Pure。我选风格 3，大众所爱的风格，彩虹 Rainbow，则输入 3。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-01.png" >}}

2. 字符集设置，毫无疑问，配置 unicode，选择 1。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-02.png" >}}

3. 提示符显示当前时间风格。我选择 1，只显示命令耗时，不显示当前时间。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-03.png" >}}

4. 提示符分隔符，也就是 src/master 之间的符号风格。我钟爱箭头，选择 1 -> Angled.

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-04.png" >}}

5. 提示符头部风格，就是 `master >` 的风格选择。毫无疑问，我钟爱箭头，选择 1 - Sharp。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-05.png" >}}

6. 提示符尾部风格，钟爱箭头，但不是双箭头，选择 1 - Flat。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-06.png" >}}

7. 提示符高度，显示一行还是两行，体验过两行，还是一行更紧凑一些，选择 1 - one line。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-07.png" >}}

8. 两个命令键的间距，我喜欢两行离的近一点，选择 1 - Compact。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-08.png" >}}

9. 提示符 Icons，多点 Icon 更帅，否则看起来就和一般主题没区别了，so 选择 2 - Many Icons。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-09.png" >}}

10. 提示符丰富度，增加一些文本描述，帮助理解提示符中字符含义。还是简洁为美，毕竟空间不能占用太多，而且含义简单，无需文本辅助。我选择 1 - Consice。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-10.png" >}}

11. 这是什么配置？提示符瞬闪？好像命令执行后提示符就立刻消失，只保留在最新的提示符，先选择 n - No 看看效果吧。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-11.png" >}}

12. 提示符高性能模式，是否启用。推荐启用，就启用吧 1-verbose，如果发现有兼容问题，在重新配置 off。

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-12.png" >}}

到此基本全部的配置都已经完成，powerlevel10k 命令行提示符的最终效果，如下所示：

{{<image "2023-10/2023-10-20-zsh-theme-powerlevel10k-13.png" >}}

配置完成后，如果希望重新配置，执行 `p10k configure` 会重新打开配置向导。

## 配置文件

通过 powerlevel10k 的配置导航能快速自定义提示符的主题风格，但如想更细粒度的配置，可直接在 `$HOME/.p10k.zsh` 配置，配置导航只是最粗粒度的配置方式。

如配置提示符两侧内容，通过 `~/.p10k.sh` 中的变量 `POWERLEVEL9K_LEFT_PROMPT_ELEMENTS` 和 `POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS` 设置。

如下所示：

```zsh
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
  os_icon                 # os identifier
  dir                     # current directory
  vcs                     # git status
  # prompt_char           # prompt symbol
)

typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
  status                  # exit code of the last command
  command_execution_time  # duration of the last command
  ...
  # time                  # current time
  ...
)
```

如前面配置的时候，设置了不显示当前时间，现在可以通过打开 `POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS` 中的 `time` 就可重新显示命令的时间。

还有前面对提示符瞬闪的配置，transient 设置为 off。如果希望启用，同时有不重新经历一次配置向导，直接进入 `~/.p10k.zsh` 配置变量 `POWERLEVEL9K_TRANSIENT_PROMPT=always`，即可。

```zsh
# Transient prompt works similarly to the builtin transient_rprompt option. It trims down prompt
# when accepting a command line. Supported values:
#
#   - off:      Don't change prompt when accepting a command line.
#   - always:   Trim down prompt when accepting a command line.
#   - same-dir: Trim down prompt when accepting a command line unless this is the first command
#               typed after changing current working directory.
typeset -g POWERLEVEL9K_TRANSIENT_PROMPT=always
```

## 总结

本文介绍了 powerlevel10k 的安装与配置，它的更多能力还可以在 `~/.p10k.sh` 继续探索，诸如对诸多其他工具的支持。

最后，如果希望你命令行提示能给你提供更多信息，请安装 powerlevel10k！

我的博文：[我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
