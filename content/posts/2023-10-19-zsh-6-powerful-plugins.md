---
title: "介绍 6 个强大的 zsh 插件"
date: "2023-10-18T18:36:55+08:00"
draft: false
comment: true
tags: ["zsh"]
---

今天，将会在上一个教程的基础上，再介绍六个插件，其中 4 个是 oh-my-zsh 的内置插件，还有两个第三方插件。

让我们正式开始。

## 推荐一个网站

首先呢，在开始前，我想先推荐一个 github 仓库，awesome-zsh-plugins。直接通过浏览器打开地址 [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins)，里面提供了相当丰富的 zsh 的框架、教程、插件与主题等等。

框架，如 oh-my-zsh，还有其他的一些框架。其中，还有关于 zsh 的教程。

插件，上个视频介绍过的两个插件，zsh-syntax-highlighting - 命令行语法高亮插件, zsh-autosugggestions - 命令行自动建议提示插件，在这个文档里面都能找到。

主题，除了 oh-my-zsh，还有更多的主题可选择，例如将在后面讲介绍的 powerlevel10k 这个 zsh 主题，在这个文档里也能找到。

下面正式开始今天的内容。

## 推荐插件

先说 oh-my-zsh 的内置插件。

打开 zsh 配置文件 ~/.zshrc 文件，将要使用的 oh-my-zsh 的插件提前全部配置。

```bash
plugins=(... copypath copyfile copybuffer sudo ...)
```

保存退出。

### copypath 插件

[copypath](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copypath) 的用途如其名，就是用来 copy 路径的。

两种用法：

- copypath: 无参数，直接拷贝当前路径；
- copypath <文件或目录>：拷贝指定文件或目录的绝对路径；

相比于 `pwd` 之后再拷贝，真的是省心省力的方式。

### copyfile

[copyfile](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copyfile)，用于拷贝文件内容，命令格式 copyfile <文件路径>

一个测试命令，`copyfile test.txt`，即可将 `test.txt` 文件中的内容拷贝到剪贴板中。

### copybuffer

[copybuffer](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copybuffer)，是用于快速复制当前命令行的输入。

如何使用呢？

要通过 CTRL+o 快捷键实现命令行内容的拷贝。

> 注意：测试的时候，发现与 vi-mode 存在冲突，不过 vi-mode 已经可使用 yy，无续开启个插件；

## 其他插件

介绍完 oh-my-zsh 的内置插件，继续介绍两个三方插件，分别是 zsh-history-substring-search 和 you-should-use.

### history-substring-search

插件名称为 [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)，用于

下载命令如下：

```bash
 git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
```

下载完成之后，还将插件的能力配置到指定按键上，即要进行按键的绑定。

**方向键**

```zsh
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
```

**CTRL+P/N**

```zsh
bindkey '^p' history-substring-search-up
bindkey '^n' history-substring-search-down
```

**Vi 模式**
```zsh
bindkey -M vicmd 'k' history-substring-search-up
bindkey -M vicmd 'j' history-substring-search-up
```

### you-should-use

[you-should-use](https://github.com/MichaelAquilina/zsh-you-should-use) 用途是，如果执行的命令存在别名，会自动提示推荐使用的别名；

下载命令：

```bash
git clone https://github.com/MichaelAquilina/zsh-you-should-use.git $ZSH_CUSTOM/plugins/you-should-use
```

由于，默认的提示信息在命令输出之前，可通过 export YSU_MESSAGE_POSITION="after" 配置提示信息在命令最后输出。

## 总结

本文介绍了 6 个 zsh 插件。

copypath、copyfile 和 copybuffer 可快速拷贝特定内容到粘贴板。sudo 是一个快捷操作，通过快捷键即可将 sudo 增加命令开头。zsh-history-substring-search 插件能快速匹配历史记录命令。而最后的 zsh-you-should-use，真的是强烈推荐，特别是对于配置了类似 oh-my-zsh 这样的框架，加载一些默认能力，可能存在不少易于使用的别名，通过这个插件，就有了能力发现这些潜在的能力。

