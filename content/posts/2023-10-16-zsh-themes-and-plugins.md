---
title: "终端环境：zsh 、oh-my-zsh、提示符主题与 7 效率插件"
date: "2023-10-16T15:00:06+08:00"
draft: false
comment: true
tags: ["zsh"]
description: "本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件"
---

{{< video bb_id=619786097 yt_id=1zJcH4hZW4o >}}

前文中，对iTerm2 已经有了一个大概认识。但一个高效的终端环境，离不开一个优秀 shell 解释器。

本篇文章将主要介绍 zsh + oh-my-zsh 的安装、提示符主题配置，以及介绍 7 提升效率的 zsh 插件。

## 为什么使用 zsh？

开始前，先问为什么，知其然，要知其所以然，是个好习惯。

所以，为什么要用 zsh呢？

大家最熟悉的 shell 解释器，肯定是 bash。zsh（Z Sehll）相对于 bash（Bourne Again Shell）相对有哪些优势呢？

### 改进的自动补全能力

zsh 提供了更强大、更灵活的自动补全功能。它不但可以自动补全命令，设置选项、参数甚至文件名，都可自动补全。对于命令参数，zsh甚至可以显示简短的帮助信息，这使得探索新命令变得更加容易。

### 更好的脚本和插件支持

zsh 有一个强大的社区，提供了大量的插件和主题，如 oh-my-zsh 这个流行的 zsh 框架，允许我们轻松添加、更新插件和主题。这些插件可以增强 shell 的功能，提供便捷的别名、函数以及其他有用的特性。

本文将会介绍 oh-my-zsh 的安装过程。

### 高级的主题和提示符定制

zsh 还允许用户对命令行提示符进行高度定制，包括颜色、内容和格式。用户可以非常容易地调整提示符来显示 git 分支、Python 虚拟环境等信息。

我们会在后续介绍一款非常强大的 zsh 插件，名为 powerlevel10k，它支持完全的主题自定义特性，非常强大。

### 更智能的命令行交互

zsh 还支持 bash 不具备的一些智能特性，如拼写校正和近似完成。如果用户输入的命令有拼写错误，zsh 可以建议正确的命令。

如我输入 lls，会提示我 "zsh: correct 'lls' to 'ls' [nyae]?"

```zsh
❯ lls
zsh: correct 'lls' to 'ls' [nyae]?
```

输入 y 接受纠正建议。

当然这个是要做个简单的配置，通过 `setopt CORRECT_ALL` 启用。

### 其他

其他还有很多强大特性。如：

zsh 的命令行历史是终端间共享的，通过自动补全，能一步增强了操作效率与体验。

zsh 的文件匹配和通配符功能确实比 Bash 要强大得多，除了常规的通配符能力，还提供了一些扩展通配符、限定符等，如递归匹配 `**/`，`ls **/*.go` 会列出所有的 Go 文件。`!{pattern}`，匹配不符合模式的内容。其他更多自行探索。

zsh 的可配置性更强，zsh 提供了比 bash 更多的选项和特性，我们都可通过配置文件调整。

由于，本文还是注重实践，比较的部分就先写这么多。

> 系列阅读：
>
> - [终端环境：iTerm2](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [终端环境：zsh 与 oh-my-zsh 的 7 效率插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [终端环境：高效 shell 命令（一）之目录文件命令 - exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...

## 安装


当然如果是更早版本，或其他 Linux 发行版本，则这个步骤无法跳过。

对于不同系统，zsh 的安装命令，如下所示： 


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

对于 macOS 系统的用户，MacOS 的默认 shell 从 2019 开始以前替换为 zsh，该步骤可省略。可阅读：[What is Zsh? Should You Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in.) 其中有介绍为什么 2019 macOS 将默认的 shell 从 bash 切换到 zsh。

我看下来，主要原因就是版权问题啦。

如果你是个老古董，还是用 MacOS 2019 之前的系统，可通过如下命令安装：

```bash
brew install zsh
```

安装完成后，将 zsh 设置为默认 shell，命令如下所示：

```bash
chsh -s /bin/zsh
```

通过如下命令检查下是否成功。

```bash
echo $SHELL
zsh
```

## oh-my-zsh

[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) 是用于管理 zsh 配置的轻量级框架，具有开箱即用的特点，而且它提供了大量内置插件。让我们用它快速配置 zsh 吧！

oh-my-zsh 这个名字起的很骚气的，大概就是下面这样表情。

安装命令，如下所示：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装后，就已经有一些默认效果，如命令行提示符的主题变化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-16-zsh-themes-and-plugin-01.png)

### 主题

oh-my-zsh 提供了许多内置主题，可查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)。

我们打开 ~/.zshrc 配置文件，可执行更新主题配置：

```bash
ZSH_THEME="agnoster"` # 默认为 robbyrussell
```

执行 `source ~/.zshrc` 生效配置，就能看到主题效果。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-02.png)

另外，oh-my-zsh 还提供了 random 主题，它会在 oh-my-zsh 内置主题中随机选择一个主题展示。

编辑 `~/.zshrc`，配置如下：

```bash
ZSH_THEME="random"
```

演示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-03.gif)

说实话，我觉得没人会这么用吧。这明显很鸡肋的功能啊。

### 内置插件

重点来了，接下来我们一起来看看 zsh 的效率神器 - 插件能力吧。我先给大家推荐 7 款常用的插件，其中 5 个是 oh-my-zsh 的内置插件。下期会再给推荐 6 个插件。

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

