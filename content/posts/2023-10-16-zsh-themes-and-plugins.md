---
title: "我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件"
date: "2023-10-16T15:00:06+08:00"
draft: false
comment: true
tags: ["zsh"]
description: "本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件"
---

{{< video bb_id=619786097 yt_id=1zJcH4hZW4o >}}

本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件。

## 为什么使用 zsh？

zsh vs bash 的一些优势，如下所示：

- zsh 的补全能力强大，bash 的 Tab 补全从头匹配，如 mn 匹配 mnt，而非 findmn，而 zsh 可匹配 mnt 和 findmn； 

- zsh 的命令行历史是在 terminal 间共享，结合自动补全，进一步增强了用户体验；
- zsh 还提供自动纠错能力，如果你输入太快，它能智能给你一个可能正确的建议；
- zsh 的配置能力更强，支持构建更精美的提示主题；
- zsh 参数的扩展能力比 Bash 更强大；
- zsh 有大量插件、主题、框架，如 oh-my-zsh 框架，能助你快速配置出一个强大终端；

> 系列阅读：
>
> - [我的终端环境：iTerm2 的安装与体验](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [我的终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)

## 安装

对于 macOS 用户，zsh 从 2019 已经是默认 shell，无需安装。该步骤可省略。推荐阅读：[What is Zsh? Should You Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in.)介绍了为什么推荐使用 zsh，以及为什么从 2019 macOS 将默认的 shell 从 bash 切换到 zsh。

当然如果是更早版本，或其他 Linux 发行版本，则这个步骤无法跳过。

不同系统的安装命令如下： 

```bash
# MacOS 系统
brew install zsh
```
```bash
# Debian
apt install zsh
```
```bash
# Centos
yum install -y zsh
```
```bash
# Arch Linux
pacman -S zsh
```
```bash
# Fedora
dnf install zsh
```

安装完成，要将 zsh 设置为默认 shell，命令如下所示：

```bash
chsh -s /bin/zsh
```

通过如下命令：

```bash
echo $SHELL
zsh
```

检查是否设置成功。

## oh-my-zsh

[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) 是用于管理 zsh 配置的轻量级框架，具有开箱即用的特点，而且它提供了大量内置插件。让我们用它快速配置 zsh 吧！

安装命令如下：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装后，就已经有一些默认效果，如命令行提示符的主题变化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-01.png)

### 主题

oh-my-zsh 提供了许多内置主题，查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)。

打开 ~/.zshrc 配置文件，更新主题配置：

```bash
ZSH_THEME="agnoster"`
```

执行 `source ~/.zshrc` 生效，查看主题效果。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-02.png)

另外，oh-my-zsh 还提供了 random 主题，它会在 oh-my-zsh 内置主题中随机选择主题展示。

编辑 `~/.zshrc`，配置如下：

```bash
ZSH_THEME="random"
```

演示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-03.gif)

### 内置插件

oh-my-zsh 提供的所有内置插件，都可以在仓库 [ohmyzsh/ohmyzsh/plugins](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins)  中找到。本教程将要介绍的 oh-my-zsh  内置插件，如下所示：

- [git](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)，内置插件
- [web-search](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/web-search)，搜索引擎搜索；
- [jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools)，用于处理 json 数据；
- [z](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/z)，目录快速跳转；
- [vi-mode](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode)，使用 vi 模式编辑命令行；

#### 插件 1 - git

这个插件提供了 git 命令的大量别名，查看[git 插件文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

如一些常用命令的别名：

cmd           | alias
------------- | ------------
git clone     | gcl
git status    | gst
git commit    | gc
git add       | ga
git add --all | gaa
git diff      | gd
git push      | gp
git pull      | gl

更多命令，可自行查看[文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

#### 插件 2 - web-search

[web-search](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/web-search/) 提供在终端直接搜索信息能力，将自动跳转浏览器，到指定的搜索引擎执行搜索请求。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-04.gif)

常见的搜索引擎基本都是支持的，诸如 google, bing, baidu, 甚至是 github 等。

这个插件一般我很少用，因为我已经安装了另外一个工具 alfred（用于替代 mac 默认的 spotlight），通常都是通过它直接启动搜索，。

#### 插件 3 - jsontools

[jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools/) 提供了一些用于操作 json 数据的命令，如:

- pp_json 格式化 json；
- is_json 判断是否是 json；

演示效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-05.gif)

#### 插件 4 - z

[z 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/z) 可用于快速的目录跳转，我觉得大部分人在使用 Linux 都被它的目录跳转烦恼过。z 就是这个烦恼的救星。

想查看更多信息可找 [z 原仓库](https://github.com/agkozak/zsh-z) 查看。

介绍它的用法：

1. 输入 z，紧跟 tab 键，会直接列出访问过的目录，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-06.gif)

2. 输入 z blog，紧跟 tab 键，会直接列出访问过包含 blog 的目录，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-07.gif)

3. 输入 z tmux，因为包含 tmux 的目录只有一个，直接选中，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-08.gif)

4. 输入 z tmux，直接 Enter 确认，进入到目录，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-09.gif)

#### 插件 5 - vi-mode

[vi-mode 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode) 支持在命令行开启 vi 模式，利用 vi 键进行命令行编辑。这个插件，视个人情况，是否使用吧。如果你是一个 vi 忠实用户，可考虑开启。否则，还是简单最好，否则容易影响心情。

这个插件就不多介绍了，更多查看 [它的文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode)。

### 三方插件

将介绍 2 个非 oh-my-zsh 内置插件，即 zsh-syntax-highlighting 和 zsh-autosuggestions。之前的演示，我其实已经把这个插件已经开启了。

开始介绍前，让我们先将这两个插件全部安装配置完成。

下载命令如下所示：

```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

打开 `.zshrc` 完成配置：
    
```zsh
plugins=(git web-search jsontools z vi-mode zsh-syntax-highlighting zsh-autosuggestion)
```

记得执行 `source ~/.zshrc` 生效配置。

#### 插件 6 - zsh-syntax-highlighting

zsh-syntax-highlighting 是 zsh 的语法高亮插件，如果输入的命令不存在，或存在明显的错误，将会自动以红色表示。

演示效果，如下所示：

错误命令提示

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-10.gif)

正确命令提示

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-11.gif)

#### 插件 7 - zsh-autosuggestions

zsh-autosuggestions 用于提示补全建议，当输入字符后，它会自动给我们一些建议。输入 -> 右方向键可将建议直接输入终端。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-12.gif)

如果想改变接受建议的默认按键，例如，希望通过输入 `Ctrl + /` 接受建议，配置实现，如下所示：

```
# <Ctrl+/> 接受 auto-suggestion 的补全建议
bindkey '^_' autosuggest-accept
```

另外，如果希望 zsh-autosuggestion 不仅支持 history，也支持自动补全的建议提示， 即使如子命令、命令选项、目录文件等提示，可增加配置，如下所示：

```zsh
export ZSH_AUTOSUGGEST_STRATEGY=(history completion)
```

## 总结

本教程主要介绍了 zsh 的安装，以及常用主题和插件的配置，其中介绍的 7 个插件，希望能给大家的日常工作提供一定的帮助。

我的博文：[我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://www.poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)

