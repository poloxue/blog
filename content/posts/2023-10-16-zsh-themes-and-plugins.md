---
title: "我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件"
date: "2023-10-16T15:00:06+08:00"
draft: false
comment: true
tags: ["zsh"]
---

{{< video bb_id=619786097 yt_id=1zJcH4hZW4o >}}

本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件。

## 安装

对于 macOS 用户，zsh 从 2019 已经是默认 shell，无需安装。该步骤可省略。推荐阅读：[What is Zsh? Should You Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in.)介绍了为什么推荐使用 zsh，以及为什么从 2019 macOS 将默认的 shell 从 bash 切换到 zsh。

当然如果是更早版本，或其他 Linux 发行版本，则这个步骤无法跳过。

不同系统的安装命令如下： 

MacOS 系统

```bash
brew install zsh
```

Debian

```bash
apt install zsh
```

Centos

```bash
yum install -y zsh
```

Arch Linux

```bash
pacman -S zsh
```

Fedora

```bash
dnf install zsh
```

安装完成，要将 zsh 设置为默认 shell，命令如下所示：

```bash
chsh -s /bin/zsh
```

通过如下命令：

```bash
echo $SHELL
```

检查是否设置成功。

## oh-my-zsh

oh-my-zsh 是zsh 用于管理配置的轻量级框架。下面将用它快速配置 zsh。

安装命令如下：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 主题

oh-my-zsh 提供了许多内置主题，查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)。

打开 ~/.zshrc 配置文件，更新主题配置：

```bash
ZSH_THEME="agnoster"`
```

执行 `source ~/.zshrc` 生效，查看效果。

配置插件

### 内置插件

oh-my-zsh 提供的所有内置插件，都可以在仓库 https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins 发现。

本教程将要介绍的插件如下：

- git，内置插件
- web-search，搜索引擎搜索；
- jsontools，用于处理 json 数据；
- z，目录快速跳转；
- vi-mode，使用 vi 模式编辑命令行；

**git**

提供了 git 命令的大量别名，查看[git 插件文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

**web-search**

在终端进行信息搜索，将自动跳转到 search engine 完成搜索请求。

**jsontools**

提供了一些用于操作 json 数据的命令，如 pp_json 格式化 json，

**z**

用于快捷的目标跳转，我觉得大部分人在使用 Linux 都被它的目录跳转烦恼过。z 就是这个烦恼的救星。

**vi-mode**

支持在命令开启 vi 模式进行命令行编辑。视个人情况，是否使用吧。如果你是一个 vi 忠实用户，建议开启这个插件。否则，还是简单最好。

### 三方插件：

将介绍 2 个非 oh-my-zsh 内置插件，即 zsh-syntax-highlighting 和 zsh-autosuggestions。

提前全部安装

```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

打开 `.zshrc` 完成配置：
    
```zsh
plugins=(git web-search jsontools z vi-mode zsh-syntax-highlighting zsh-autosuggestion)
```

记得执行 `source ~/.zshrc` 生效配置。

**zsh-syntax-highlighting**

zsh-syntax-highlighting 是 zsh 的语法高亮插件，如果输入的命令不存在，或存在明显的错误，将会自动以红色表示。

**zsh-autosuggestions**

zsh-autosuggestions 用于提示补全建议，当输入字符后，它会自动给我们一些建议。输入 -> 右方向键可将建议直接输入终端。

如果希望改变接受建议的默认按键，可通过类似如下配置实现：

```
# <Ctrl+/> 接受 auto-suggestion 的补全建议
bindkey '^_' autosuggest-accept
```

## 总结

本教程主要介绍了 zsh 的安装，以及常用主题和插件的配置，其中介绍的 7 个插件，希望能给大家的日常工作提供一定的帮助。
