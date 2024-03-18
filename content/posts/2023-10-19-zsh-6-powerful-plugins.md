---
title: "终端环境：6 个强大的 zsh 提效插件"
date: "2023-10-18T18:36:55+08:00"
draft: false
comment: true
tags: ["zsh"]
description: "今天，将会在上文的基础上，再介绍 6 个插件，其中 4 个是 oh-my-zsh 的内置插件，还有两个第三方插件。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-08.png)

本文是高效终端环境第三篇，介绍 6 个可用于提效 zsh 效率的插件。系列查看：[我的终端环境](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MzE2NTY2MA==&action=getalbum&album_id=3299748150420979713#wechat_redirect)

视频教程：

{{< video bb_id=322450976 yt_id=RSaiddMI6Ns >}}

今天，将会在 [上文](https://www.poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/) 的基础上，再介绍六个插件，其中 4 个是 oh-my-zsh 的内置插件，还有两个第三方插件。

## 快速一览

本文将会涉及的插件，如下所示：

- copypath，拷贝路径；
- copyfile，拷贝文件内容；
- copybuffer，拷贝命令行内容；
- sudo，快捷 sudo，命令行快捷添加 sudo 插件；
- zsh-history-substring-search，命令历史记录子字符串匹配；
- zsh-you-should-use，用于命令行 alias 别名提醒；

让我们正式开始。

## 推荐一个网站

在开始前，我想先推荐一个 github 仓库，awesome-zsh-plugins，通过浏览器打开 [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins)，里面提供了相当丰富的 zsh 的框架、教程、插件与主题等等，是 zsh 的资源合集。

框架，如 oh-my-zsh，还有其他的一些框架。其中，还有关于 zsh 的教程。

插件，上个视频介绍过的两个插件，zsh-syntax-highlighting - 命令行语法高亮插件, zsh-autosugggestions - 命令行自动建议提示插件，在这个文档里面都能找到。

主题，除了 oh-my-zsh 内置主题，还有更多主题可选，如将在后面讲介绍的 powerlevel10k 这个 zsh 主题，在这个文档里也能找到。

## 推荐插件

先说 oh-my-zsh 的内置插件。

打开 zsh 配置文件 ~/.zshrc，将要使用的 oh-my-zsh 的内置插件提前配置。

```bash
plugins=(... copypath copyfile copybuffer sudo ...)
```

保存退出，执行 `source ~/.zshrc 生效`。

### copypath

[copypath](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copypath) 的用途如其名，就是用来 copy 路径的。

支持两种用法。

copypath: 无参数，直接拷贝当前路径；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-01.gif)

copypath <文件或目录>：拷贝指定文件或目录的绝对路径；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-02.gif)

相比于 `pwd` 之后再拷贝，这种方式真的是省心省力的方式。

### copyfile

[copyfile](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copyfile)，用于拷贝文件内容，命令格式 copyfile <文件路径>。

假设，现有一个文件 test.txt。

```bash
cat test.txt
Hello oh my zsh
```

一个测试命令，`copyfile test.txt`，即可将 `test.txt` 文件中的内容拷贝到剪贴板中。

效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-03.gif)

无需鼠标选中复制粘贴。

### copybuffer

[copybuffer](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copybuffer)，是用于快速复制当前命令行的输入。

如何使用呢？

它不同于前面两个快捷键，要通过 CTRL+o 快捷键拷贝命令行内容。

特别说明，我在测试的时候，发现 copybuffer 与 vi-mode 存在冲突，不过如果启用了 vi-mode， 命令行内容拷贝可直接使用 yy，无续开启 copybuffer；

### sudo 

sudo 的主要作用是，当我们输入某个命令，如 vim /etc/zshrc，发现没有系统权限，利用 sudo 插件，可快速将 sudo 作为前缀添加到命令最前面。

演示效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-06.gif)

## 其他插件

介绍完 oh-my-zsh 的内置插件，继续介绍两个三方插件，分别是 zsh-history-substring-search 和 you-should-use.

将 zsh-history-substring-search 和 zsh-you-should-use 两个插件下载配置。

```bash
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
git clone https://github.com/MichaelAquilina/zsh-you-should-use.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/you-should-use
```

打开 `~/.zshrc` 文件，更新如下内容：

```zsh
plugins=(... zsh-history-substring-search you-should-use)
```

### history-substring-search

先介绍 [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)。它的主要用途是什么？

一般情况下，在使用 zsh 时，通过 ↑ 或 ↓ 方向键，能实现类似按前缀匹配补齐的效果。

而如果输入的是中间的字符串，则没法自动补齐。这个插件真是为这个目的而生的。

使用这个插件前，除了启用插件以外，还需要进一步配置下，将 zsh-history-substring-search 提供的能力绑定到快捷按键。

例如，上下方向键 ↑ 和 ↓。

```zsh
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
```

在生效配置后，测试失败的话，查看文档，其中有介绍：

> However, if the observed values don't work, you can try using terminfo:
>
> bindkey "$terminfo[kcuu1]" history-substring-search-up
> bindkey "$terminfo[kcud1]" history-substring-search-down

那我们就增加这两行配置吧。

```zsh
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down
```

除了 ↑ ↓ 按键外，我一般还习惯使用 CTRL+P/N 上下查找历史记录，配置如下：

```zsh
bindkey '^p' history-substring-search-up
bindkey '^n' history-substring-search-down
```

如果希望支持 vi 的 jk，配置如下：

```zsh
bindkey -M vicmd 'k' history-substring-search-up
bindkey -M vicmd 'j' history-substring-search-up
```

保存生效配置，测试下最终的成功成果吧。效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-04.gif)

另外，高亮样色可配置化的，可通过类似如下语法实现：

```
export HISTORY_SUBSTRING_SEARCH_HIGHLIGHT_FOUND=(bg=none,fg=magenta,bold)
```

设置 background 为 none，即无色，而 front 设置为 magenta,bold。效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-07.png)

如上的 zsh 的颜色变量，可查看 [zsh 仓库文档](https://github.com/zsh-users/zsh/blob/master/Functions/Misc/colors) 发现更多颜色。

```zsh
color=(
# Codes listed in this array are from ECMA-48, Section 8.3.117, p. 61.
# Those that are commented out are not widely supported or aren't closely
# enough related to color manipulation, but are included for completeness.

# Attribute codes:
  00 none                 # 20 gothic
  01 bold                 # 21 double-underline
  02 faint                  22 normal
  03 italic                 23 no-italic         # no-gothic
  04 underline              24 no-underline
  05 blink                  25 no-blink
# 06 fast-blink           # 26 proportional
  07 reverse                27 no-reverse
# 07 standout               27 no-standout
  08 conceal                28 no-conceal
# 09 strikethrough        # 29 no-strikethrough

# ...

# Bright color codes (xterm extension)
  90 bright-gray            100 bg-bright-gray
  91 bright-red             101 bg-bright-red
  92 bright-green           102 bg-bright-green
  93 bright-yellow          103 bg-bright-yellow
  94 bright-blue            104 bg-bright-blue
  95 bright-magenta         105 bg-bright-magenta
  96 bright-cyan            106 bg-bright-cyan
  97 bright-white           107 bg-bright-white
)
```

### you-should-use

[you-should-use](https://github.com/MichaelAquilina/zsh-you-should-use) 用途是，如果执行的命令存在别名，会自动提示推荐使用的别名；

由于，默认的提示信息在命令输出之前，添加如下配置：

```bash
export YSU_MESSAGE_POSITION="after"
```

它的作用是，实现将提示信息打印在命令输出的最后。

最终效果演示，如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-05.gif)

## 总结

本文介绍了 6 个 zsh 插件，每个插件都有特定的场景用途，希望能给大家的日常工作提升效率。

