---
title: "终端环境：zsh 、oh-my-zsh、提示主题与 7 效率插件"
date: "2023-10-16T15:00:06+08:00"
draft: false
comment: true
tags: ["zsh"]
description: "本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-17.png)

本文是高效终端环境第二篇，系列查看：[我的终端环境][https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MzE2NTY2MA==&action=getalbum&album_id=3299748150420979713#wechat_redirect]

视频版本，没有文章详细：

{{< video bb_id=619786097 yt_id=1zJcH4hZW4o >}}

前文中，对 iTerm2 已经有了一个大概认识。但一个高效的终端环境，离不开一个优秀 shell 解释器。

本篇文章将主要介绍 zsh + oh-my-zsh 的安装、提示符主题配置，以及介绍 7 提升效率的 zsh 插件。

## 为什么使用 zsh？

开始前，先问为什么，知其然，要知其所以然，是个好习惯。

所以，为什么要用 zsh 呢？

大家最熟悉的 shell 解释器，肯定是 bash。zsh（Z Sehll）相对于 bash（Bourne Again Shell）相对有哪些优势呢？

### 改进的自动补全能力

zsh 提供了更强大、更灵活的自动补全功能。它不但可以自动补全命令，设置选项、参数甚至文件名，都可自动补全。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-18.gif)

对于命令参数，zsh 甚至可以显示简短的帮助信息，这使得探索新命令变得更加容易。

### 更好的脚本和插件支持

zsh 有一个强大的社区，提供了大量的插件和主题，如 oh-my-zsh 这个流行的 zsh 框架，允许我们轻松添加、更新插件和主题。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-21.gif)

这些插件可以增强 shell 的功能，提供便捷的别名、函数以及其他有用的特性。

### 高级的主题和提示符定制

zsh 还允许用户对命令行提示符进行高度定制，包括颜色、内容和格式。用户可以非常容易地调整提示符来显示 git 分支、Python 虚拟环境等信息。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-19.png)

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

如果你想深入学习 zsh，推荐 [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins) 这个仓库。或者推荐看一个关于 zsh 的深度系列文章：

- [Configuring Zsh Without Dependencies](https://thevaluable.dev/zsh-install-configure-mouseless/)
- [A Guide to Zsh Expansion with Examples](https://thevaluable.dev/zsh-expansion-guide-example/)
- [A Guide to the Zsh Completion with Examples](https://thevaluable.dev/zsh-completion-guide-examples/)
- [A Guide to the Zsh Line Editor with Examples](https://thevaluable.dev/zsh-line-editor-configuration-mouseless/)

好吧，前导部分写的有点长。本文还是注重实践，比较的部分就先写这么多。

## 安装

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

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-13.gif)

想表达的可能是，当别人看你用 oh-my-zsh 配置的终端，大概率发出 "wow! 你的终端太赞了！"

OK，那下面让我们尝试让它赞起来吧。

## 安装

首先，oh-my-zsh 的安装很简单。

安装命令，如下所示：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装后，就已经有一些默认效果，如命令行提示符的主题变化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-01.png)

这是默认的  oh-my-zsh 主题 "robbyrussell"。

## 主题

oh-my-zsh 提供了许多内置主题，可查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) 获取一系列的主题。

我们可直接通过 ~/.zshrc 配置更新主题配置，将内容修改如下：

```bash
ZSH_THEME="agnoster"` # 默认为 robbyrussell
```

执行 `source ~/.zshrc` 生效配置，就能看到主题效果。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-02.png)

另外，oh-my-zsh 还提供了 random 主题，它会在 oh-my-zsh 内置主题中随机选择一个主题展示。只需编辑 `~/.zshrc`，将 ZSH_THEME 更新为 random 即可。

配置如下所示：

```bash
ZSH_THEME="random"
```

演示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-03.gif)

说实话，我觉得没人会这么用吧。这明显很鸡肋的功能啊。

## 内置插件

重点来了，接下来我们一起来看看 zsh 的效率神器 - 插件能力吧。

我先给大家推荐 7 款常用的插件，其中 5 个是 oh-my-zsh 的内置插件。考虑内容不宜太长，下期会再推荐 6 个插件。

oh-my-zsh 提供的所有内置插件，都可以在仓库 [ohmyzsh/ohmyzsh/plugins](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins)  中找到，每个插件都有相应的介绍文档。

本教程将要介绍的 5 个 oh-my-zsh  内置插件，如下所示：

- [git](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)，Git 插件，其实就是提供一些常用的 git 命令别名。
- [web-search](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/web-search)，命令行打开搜索引擎，已支持大部分搜索引擎；
- [jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools)，用于格式化 json 数据；
- [z](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/z)，基于历史访问目录的快速跳转；
- [vi-mode](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode)，使用 vi 模式编辑命令行；

启用所有插件，打开 `zshrc` 配置，把这些内置插件都打开，如下所示：

```bash
plugins=(git web-search jsontools z vi-mode)
```

### 插件 1 - git

Git 插件提供了 git 命令的大量别名，查看[git 插件文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

如下一些常用命令的别名：

```bash
git clone     -> gcl
git status    -> gst
git commit    -> gc
git add       -> ga
git add --all -> gaa
git diff      -> gd
git push      -> gp
git pull      -> gl
```

更多命令的映射关键关系，可自行查看它的[文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

这个插件不错，但有个缺点，这么多可用别名，我又记不住，岂不是成了摆设。如果想用好，我每次都用去查文档吗？不查文档行不行呢？

当然也是可以的，oh-my-zsh 中启用的一些其他插件可能也会有别名。

其实，有一个插件可帮忙我们解决这个问题，叫做 `you-should-use`，这是下期要介绍的一个插件。简单说下，它的作用是，当我们输入一个命令时，如果这个命令存在别名，它会提示我们要使用别名。

### 插件 2 - web-search

[web-search](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/web-search/) 提供了在终端直接搜索信息的能力。

当然，其实也不是完全在终端完成，它会自动跳转浏览器，转到指定的搜索引擎执行搜索请求。

效果大概就是下面这样：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-04.gif)

常见的搜索引擎基本都是支持的，诸如 google, bing, baidu, 甚至是 github 等。

不过，我也得承认，其实这个插件一般我本人很少用，因为我已经安装了另外一个工具 alfred（替代 mac 默认的 spotlight），我都是通过它直接启动搜索。

### 插件 3 - jsontools

接下的这个插件，名为 [jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools/) ，即用于 json 的 tool。其实它只提供了一些操作 json 的基本命令，如下:

- pp_json 实现 json 字符串格式化；
- is_json 判断是否是 json；

我们直接看下演示效果吧，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-05.gif)

还是得说明，如果你没有更好的方案，安装了 oh-my-zsh，这是个不错的选择，因为你可能以前都没用过这类工具。

不过其实这个插件呢？我也很少用。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-14.gif)

我习惯使用一款叫做 jq 的命令，如果你了解它，就知道它多强大。后面说到高效命令的时候，会介绍到它的。

### 插件 4 - z

[z 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/z) 可用于快速的目录跳转，我觉得大部分人在使用 Linux 都被 cd 跳转目录跳转烦恼过。

z 就是这个烦恼的救星。

想查看更多信息可找 [z 原仓库 - zsh-z](https://github.com/agkozak/zsh-z) 查看。oh-my-zsh 下的 z 文档说明中提到，它是从这个 zsh-z 的插件中拷贝而来的。

我们来介绍它的用法，简单来说，它是基于历史访问过的目录快速跳转。我们无需输入全路径，即可完成目录切换。

下面是一些实际案例。

首先，我直接输入 z，紧跟 tab 键，会看到如下的效果。它会直接将访问过的目录都列出来。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-06.gif)

这些由 tab 产生的自动补全目录都是历史访问过的目录。因为，在没有输入任何内容的情况下，我们输入 tab 的，它列出最近访问过的目录。

如果我们输入形如 z substring，即提供子字符串，它们将所有匹配 substring 的目录都列举出来。

效果如下：

例如，我们输入 z blog，紧跟 tab 键，会直接列出访问过包含 blog 的目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-07.gif)

如果输入内容只有一个关联的目录名，它会如图上一样直接补全。

演示效果：

我们输入 z tmux，因为匹配 tmux 的目录只有一个，将会被直接选中。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-08.gif)

当然，其实这里匹配的目录名只有一个，直接输入 Enter 就可以进入目录，无需 tab 选择多次一举了。

演示效果：

我们输入 z tmux，直接 Enter 确认，即可进入到目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-09.gif)

z 非常强大是吧？

其实，有一款更强大的命令，名为 zoxide，也提供了类似的能力，它的灵感是来源于 z。我一般用的是它，后面我会介绍。

当然，这不妨碍你继续使用 oh-my-zsh 内置的工具 z，毕竟它很容易配置。

### 插件 5 - vi-mode

[vi-mode 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode) 支持在命令行开启 vi 模式，利用 vi 键进行命令行编辑。这个插件，视个人情况，是否使用吧。如果你是一个 vi 忠实用户，可考虑开启。否则，还是简单最好，否则容易影响心情。

这个插件就不多介绍了，更多查看 [它的文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode)。另外，如果确实对 vim 感兴趣，也可以考虑另外一个 vi 插件，名为 [zsh-vi-mode](https://github.com/jeffreytse/zsh-vi-mode)，它的能力更强大，也解决这个默认 vi 插件的一些不好用的 bug，不过它的配置有点复杂。

## 三方插件

我们再来了解 2 个非 oh-my-zsh 内置插件，即 zsh-syntax-highlighting 和 zsh-autosuggestions。这两个插件由 zsh 社区开发。

开始介绍前，先将这两个插件全部安装配置完成。

### 下载

下载命令如下所示：

```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### 配置

打开 `.zshrc` 完成配置：
    
```zsh
plugins=(git web-search jsontools z vi-mode zsh-syntax-highlighting zsh-autosuggestion)
```

记得执行 `source ~/.zshrc` 生效配置。

### 插件 6 - zsh-syntax-highlighting

zsh-syntax-highlighting 是 zsh 的语法高亮插件，如果输入的命令不存在，或者输入 shell 语法不正确，将会自动以红色表示。它的优点就是，当我们在终端输入，实时输入实时反馈。

首先，我们尝试下错误命令，提示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-10.gif)

再来看看，正确命令提示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-11.gif)

对，就是这么简单。通过这个插件提供的实时反馈，可以防止我们在命令执行后，才知道输入错了。

### 插件 7 - zsh-autosuggestions

zsh-autosuggestions 可以说是我最喜欢的插件了。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-16.gif)

它的作用是什么呢？

它可用于提示补全建议，当输入字符，默认情况下，它基于我们的历史命令自动提供输入建议。还记得前面提到的，zsh 的历史命令是在不同的会话间共享。现在，再结合 zsh-autosuggestions 插件，简直不要太爽。哈哈。

我们先看下效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-12.gif)

默认情况下，输入右方向键 → 可将建议直接输入终端。

但这个其实体验很差，对于一个双手不想离开键盘中心区域的人而言，通过右键接受提示建议，这简直不能忍啊。是否能改变这个默认快捷键呢？

我的目标是希望通过输入 `Ctrl + /` 接受建议，配置实现，如下所示：

```
# <Ctrl+/> 接受 auto-suggestion 的补全建议
bindkey '^_' autosuggest-accept
```

对！不要怀疑，CTRL+/ 的的字符表示就是 '^_'，我们可以通过执行 cat 命令查看，输入 CTRL+/，会到看如下输出。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-15-v1.gif)

如果你不知道想要设置的快捷键的字符表示，可以通过这种方式找到。

另外，如果希望 zsh-autosuggestion 不仅支持 history，也支持自动补全的建议提示，即原来那些要输入 tab 才能出现的内容，如子命令、命令选项、目录文件等提示，也能在提示建议的范围中。我们只需增加 completeion 这个配置项。

如下所示：

```zsh
export ZSH_AUTOSUGGEST_STRATEGY=(history completion)
```

现在，如果输入时，还没有历史命令可作为建议，会提供类似于目录、参数选项等建议。

类似于如下的效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-22.gif)

这是一篇含 GIF 量很高的文章啊。

## 总结

本文想要介绍的内容已完成。我们从 zsh 与 bash 对比，了解到 zsh 的强大。接着开始实操，从安装 zsh，oh-my-zsh、主题配置，到介绍 5 个内置插件，2 个三方插件。

最后，希望本文能对你的终端操作效率提到一点点帮助，我就心满意足了。

我的博文：[我的终端环境：zsh、oh-my-zsh，提示主题和 7 个效率插件](https://www.poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)


