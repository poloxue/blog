---
title: "从 0 开始：教你如何配置 zsh"
date: 2023-09-17T15:06:02+08:00
draft: false
comment: true
tags: ["zsh"]
---

本文将介绍如何使用 zsh 来提升命令行的操作效率。

你是否每天都在与命令行打交道？

如果答案是 "Yes"，那你肯定想拥有一个强大可定制的 Shell。 而 zsh 就是为这个目标而生，它运行于诸如 Linux 、MacOS 等类 Unix 系统下，可替换默认的 bash。

## zsh 是什么？

zsh，或 Z Shell，是一个 Unix-Like 系统（如 macOS 或 Linux）下的 shell 命令行解释器。

它支持强大的自动补全能力，拥有丰富的插件，具有高可定制性，而且与 bash 充分兼容。虽然，它与 bash 相比，能力更加强大，但是它却依然比 bash 更快。

再者，相较于 bash，zsh 现在社区更加活跃，是一个还在成长中的项目。

## zsh 的优势

zsh 和 bash 都是非常流行的 Unix-like shell，它们有着很多相似的功能特性。但相对于 bash，zsh 有这些差异化优势：

- 更优秀的命令行补全能力；
- 配置化能力更强；
- 更现代的语法；
- 改进的错误报告；
- 模拟 bash；
- 用户社区不断壮大，更新频繁；
- 体验更好的按键
- 支持vi模式等

## 安装 zsh

Linux 系统下的安装非常简单。

Debian: 
```bash
sudo apt install zsh
```

Arch Linux:

```bash
sudo pacman -S zsh
```

Fedora

```bash
sudo dnf install zsh
```

> 译注：从 2019 后，macOS 的默认 shell 从 bash 更新到了 zsh，无需安装，以往版本可通过 `brew install zsh` 安装。推荐阅读：[什么是 zsh？我该使用 zsh 吗？]

### 设置zsh为默认shell

成功安装之后，可将 zsh 设置为默认 shell。执行如下命令：

```bash
chsh -s /bin/zsh
```

输入密码确认！系统默认 shell 即修改为 zsh。zsh 的配置文件位于 ~/.zshrc，开始定制专属你的 zsh 吧。

## 如何使用

如何充分使用 zsh？开始我们的一步一步配置吧。

### 1. 配置文件

让我们一起看看 zsh 配置文件吧。

- .zshrc 或者 ~/.config/zsh/.zshrc，它是在每次启动 shell 都会运行的文件；
- .zprofile，登录你的系统时运行，与 .profile 类似；
- .zlogin，与 .zprofile 类似，唯一区别在于 .zlogin 运行在 .zshrc 之后；
- .zlogout，退出登录终端时运行。它用于在退出登录时保存日志，或执行其他配置命令。但实际上，或许 cronjob 更适合这类任务；
- .zshenv，用于设置环境变量；

注意，以上所有的文件都有一个系统级别的对应文件，位于 /etc/zsh*，如 .zshrc 对应于 /etc/zshrc。通常，不同的 Linux 发行版会有自己的专属配置。

### 2. 定制 .zshrc

开始定制你的 .zshrc。路径可能位于 $HOME 或 $HOME/.config/zsh/。

首先，让我们设置一个命令行提示。

```bash
autoload -U colors && colors
PS1="%B%{$fg[red]%}[%{$fg[yellow]%}%n%{$fg[green]%}@%{$fg[blue]%}%M %{$fg[magenta]%}%~%{$fg[red]%}]%{$reset_color%}$%b "
```

将 PS1 改造和 bash 有些许不同。虽然，这个 PS1 看起来依然很复杂，但比起来 bash 的方式还是更容易些。

保存 .zshrc，重启打开 zsh 查看，如下图所示：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-20032023-01.png)

配置自动补全插件:

```bash
# Basic auto/tab complete
autoload -U compinit
zstyle ':completion:*' menu select
zmodload zsh/complist
setopt extendedglob
_comp_options+=(globdots)
```

解释下：

1. 行 1：为 zsh 启用自动补全；
2. 行 2：启用菜单的完成选项并选择（Tab将弹出菜单，回车将选择或完成命令）；
3. 行 3：加载 zsh complist（完成脚本）；
4. 行 4：启用 ** 通配符（通配符匹配任何文件/目录）
5. 行 5：启用隐藏文件的自动完成功能

保存文件，执行 `ls` 命令，输出如下：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-20032023-02.png)

### 3. 安装框架

市面上有很多由于管理 zsh 配置的框架，如 Oh-my-zsh、Prezto、Zinit 和 Antigen。 

其中，Oh My Zsh 深受用户欢迎。它配备了许多默认功能，改善您的命令行体验，如自动完成、插件、主题、语法高亮、alias 别名、自定义提示和历史命令管理。

#### 安装 oh-my-zsh

使用 curl 安装 oh-my-zsh，确保你的系统已经成功安装了 `curl`。

```bash
sudo apt install curl    #Ubuntu / Debian
sudo pacman -S curl   #Arch Linux
sudo dnf install curl   #Fedora
```

下载安装 oh-my-zsh 脚本：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

如上命令将安装 oh-my-zsh，并将现有 .zshrc 保存为 .zshrc.pre-oh-my-zsh。

安装成功后，oh-my-zsh 会覆盖你的自定义提示，效果如下所示：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-20032023-03.png)

#### 启用插件

安装成功后，$HOME 目录下将会有一个名为 `.oh-my-zsh` 目录。zsh 的内置插件都在 ~/.oh-my-zsh/plugins。

查看 oh-my-zsh 提供的插件，输入：

```bash
$ cd ~/.oh-my-zsh/plugins/
$ ls
```

启用插件也很简单，编辑 ~/.zshrc, 将插件名称追加到 `plugins=()` 中即可。

示例如下：

```bash
plugins=(golang git autocd)
```

#### Tab 补全

安装配置 oh-my-zsh 成功之后，tab 补全已经启用。如下效果：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-19032023-01.gif)

#### 提示主题

oh-my-zsh 的提示主题可提升终端视觉体验，如方便地查看当前所在目录、用户名、主机名等。这些通过内置插件即可实现。

那么，如何启用 oh-my-zsh 提示主题？

首先，你可在 `~/.oh-my-zsh/themes/` 目录或 [GitHub 主题](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) 页面上查看你想配置的主题。假设，确认选择名为 `simple` 主题，编辑 `.zshrc`，设置 `ZSH_THEME` 变量。

配置如下所示：

```bash
ZSH_THEME="simple"
```

重启打开一个新的终端，将会有提示信息的主题已经变了。

其他主题推荐：[powerlevel10k](https://github.com/romkatv/powerlevel10k/blob/master/README.md#oh-my-zsh) 和 [oh-my-posh](https://ohmyposh.dev/docs/themes)

> 译注：本人在用的就是 powerlevel10k 主题。但它不是内置主题，要单独安装。

### 语法高亮

zsh 语法高亮，广泛使用的插件之一是 [fast-syntax-highlighting](https://github.com/zdharma-continuum/fast-syntax-highlighting)。但它并非是 oh-my-zsh 内置插件，因为要手动安装。

安装命令：

```bash
git clone https://github.com/zdharma-continuum/fast-syntax-highlighting.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting
```

配置到 `.zshrc` 的插件列表，如下所示：

```bash
plugins=(golang git autocd fast-syntax-highlighting)
```

重新打开终端，查看：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-19032023-03.gif)

> 译注：zsh-syntax-highlighting 似乎比 fast-syntax-highlighting 受欢迎，一个站点提供对比信息，访问：[fast-syntax-highlighting vs zsh-syntax-highlighting](https://www.libhunt.com/compare-fast-syntax-highlighting-vs-zsh-syntax-highlighting)。

### vim 模式

启用 vim 模式，编辑.zshrc，如下：

```bash
bindkey -v
```

如上配置即可启用 vi，在终端中使用 vi 键，用 ESC 进入 vi Normal 模式，便可使用 vim 键进行导航和操作文本。

在 vi 命令和插入模式下，绑定 Ctrl+e 实现 $EDITOR 编辑当前命令：

```
bindkey -M viins '^e' edit-command-line
bindkey -M vicmd '^e' edit-command-line
```

> 译注：作者这里介绍如何自定义 vi-mode，但其实 oh-my-zsh 内置了 `vi-mode` 插件，阅读 [readme: vi-mode plugin](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode) 了解它更多功能。github 还有其他开源的 zsh-vi-mode 插件，没有去体验，猜测比内置的功能更强大。

### 历史记录

在开始 history 命令前，我们先了解下 `.zshrc` 中与 history 相关的一些变量和选项。

```bash
HISTSIZE=10000
SAVEHIST=10000
setopt appendhistory
setopt INC_APPEND_HISTORY
setopt SHARE_HISTORY
```

说明如下：

- 行 1：设置保留历史记录最大行数；
- 行 2：设置文件中历史记录最大行数；
- 行 3：使用多个 zsh 时，追加到历史文件而不是替换；
- 行 4: 直接追加历史记录而非等待 shell 退出；
- 行 5：允许使用不同终端输入的命令；如果同时使用多个终端，这个配置非常有用；

如何输出历史记录？使用 `history` 命令即可，如下所示：

```bash
history
```

注意，它不会保存任何只是输入但未执行的命令。

输出最后 10 行命令：

```bash
history -10
```

显示大于等于 20 行的命令。

```bash
history +20
```

up 方向键，可用于查看上一个命令，down 方向键可用于查看下一个命令。

`!` 是在历史命令记录查看上，有特定的能力，可协助快速使用历史命令。

让我们看一些例子：

```bash
!!<Press Tab>
```

如果在 `!!` 后按 Tab，将自动显示最后一个命令。特别是一些要超级权限才能执行的命令，忘记输入 `sudo`，可通过这种方式快速补齐。  

示例如下：

```bash
apt update
# Does not work, it needs sudo
sudo !!<Press tab>
```

按 Tab 后将自动将 `apt update` 补齐到 `sudo` ，实现快速输入。

如果将 `!` 与 <number> 结合，将也会有不一样的体验。

```bash
!<number> <press tab>
```

按下 Tab 后，输入会被会自动补全为历史记录中对应编号的命令。

### 命令补全

任何命令，如想实现 zsh 自动补全，则必须提供补全规则脚本。zsh 默认的补全规则文件存放在 `/usr/share/zsh/site-functions/` 下。如某命令无法自动补全，尝试 google 搜索该命令的补全脚本，并将其放在该目录下。

补全目录的具体位置以 `$fpath` 为准，不同系统发行版本可能有所不同。

```bash
echo $fpath
```

你也可以自定义补全目录位置，将其追加到这个变量之后即可。

### 使用 FzF

Fzf 是一个模糊匹配查找器，可与 zsh 完美配合。通过 key-binding 或使用 fzf-tab 插件实现命令或目录的补全。

让我们开始安装。

首先，这个插件要在 zshrc 的最后加载。因此，我们无法用 oh-my-zsh 来管理这个插件。

插件下载，将其 clone 到 `~/.config/zsh`，命令如下：

```bash
git clone https://github.com/Aloxaf/fzf-tab ~/.config/zsh/fzf-tab/
```

激活插件，将 fzf-tab 插件附加到 .zshrc 末尾：

```bash
echo 'source ~/.config/zsh/fzf-tab/fzf-tab.plugin.zsh' >> ~/.config/zsh/.zshrc
```

效果如下：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-19032023-02.gif)

> 译注：文中缺少了一个步骤，还需要安装 `fzf` 命令, macos 安装： `brew install fzf`。

> 译注：这个插件交互体验不错，但整体感觉下来不如 [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 操作快捷，先不启动了，大家可以试个人情况而定。另外，在测试过程中，我发现，`zsh-autosuggestions` 与 `fzf-tab` 似乎是无法同时启用的。

### 使用 zsh-autocomplete

zsh-autocomplete，受大家欢迎的自动补齐插件之一，它提供了一些高级的命令。

说明，`zsh-autocomplete` 和其他插件存在一些兼容性问题。故而，我们将用 `zsh-autocomplete` 默认提供的 `.zshrc` 配置测试。

操作前，备份当前 `.zshrc`。

```bash
# If it is in home
mv ~/.zshrc ~/zshrc
# If it is in .config/zsh
mv ~/.config/zsh/.zshrc ~/.config/zsh/zshrc
```

下载并安装插件：

```bash
# Change your directory to ~/.config/zsh first, create if not exists
cd ~/
git clone https://github.com/marlonrichert/zsh-autocomplete
mv zsh-autocomplete/.zshrc ~/
```

源码仓库中的 .zshrc，未设置插件路径。建议进入 `~/.zshrc` 手动设置或使用 sed 命令更改：

```bash
sed -i 's|/path/to|~/.config/zsh/zsh-autocomplete|g' ~/.zshrc
```

> 译注：有两个问题。
> 1. zsh-autocomplete 源码最新版目录下无 `.zshrc`，要 checkout 到 `22.01.21` tag 下，才能继续.
> 2. sed 在 macos 下存在兼容性问题，要加上 -e 选项，才能正常工作。

> 译注：读了下 [zsh-autocomplete 说明文档](https://github.com/marlonrichert/zsh-autocomplete/tree/main)，其中，没有 oh-my-zsh 的安装说明。

重新打开终端，输入查看效果：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-19032023-04.gif)

### 按键绑定

Key Bindings，即按键绑定，它让 zsh 给你带来更加丝滑的体验，诸如实现将按键绑定到特定功能、启动程序、复制粘贴等。注意，部分组合键可能无效，shell 捕捉按键的能力是有限的。

要注意的是，在 zsh 中进行按键绑定，要用 ASCII 方式表示按键。

如下代码，将 TAB 绑定到 fzf-tab 插件，从而在 vi 插入模式，通过 `Tab` 调用 `fzf` 实现补全。

```bash
bindkey -M viins '^I' fzf-tab-complete
```

> 译注：
> Ctrl+I 与 Tab 在按键效果上等通。虽然这里绑定的是 Ctrl+I，但也等同于绑定 Tab。其他类情况还有，Ctrl+M 等同于 Enter，Ctrl+H 等同于 `backspace` 等。更多查看 [bash 快捷键](https://ostechnix.com/list-useful-bash-keyboard-shortcuts/)。

oh-my-zsh 或其他插件提供的功能函数，都可以通过这种方式绑定到相应的快捷键。

### 窗口标题

Shell 一般会提供设置窗口标题的能力。如当使用如 Alt+Tab 切换窗口时，可通过标题快速发现目标。我们可以通过查看窗口标题来确认终端当前目录。

注意，窗口标题的位置，或功能上在不同环境下可能有所差异，因为每个环境有自己的实现。

有一种方法可以通过使用 .zshrc 中的函数来动态更改窗口标题。代码实现不多解释啊了，但放心在你的 .zshrc 中使用：

```zsh
case "$TERM" in (rxvt|rxvt-*|st|st-*|*xterm*|(dt|k|E)term)
    local term_title () { print -n "\e]0;${(j: :q)@}\a" }
    precmd () {
      local DIR="$(print -P '[%c]')"
      term_title "$DIR" "st"
    }
    preexec () {
      local DIR="$(print -P '[%c]%#')"
      local CMD="${(j:\n:)${(f)1}}"
      #term_title "$DIR" "$CMD" use this if you want directory in command, below only prints program name
	  term_title "$CMD"
    }
  ;;
esac
```

这段代码会将当前工作目录名作为窗口标题，效果如下：

![](https://linuxopsys.com/wp-content/uploads/2023/03/how-to-use-zsh-instead-of-bash-20032023-06.png)

> 译注：在 iTerm2 下，这段代码还需要一些特殊处理，才能生效。具体细节可查看这个回答 [Change iTerm2 window and tab titles in zsh](https://superuser.com/questions/292652/change-iterm2-window-and-tab-titles-in-zsh)



### Zsh 命令

在 zsh 下，有一些特定命令可用于提高你的工作效率。

十六进制数转十进制：

```zsh
$ echo $((16#ff))
255
```

十进制转十六进制：

```zsh
$ echo $(([##16]255))
FF
```

打印 ASCII 字符整数值：

```bash
$ echo $((#\a)) 
97
```

字符串拆分为数组：

```bash
$ echo "${(@f)$(echo hello\nworld)}"
hello world
```

自定义分隔符拆分字符串为数组：

```bash
$ echo "${(s._.)$(echo hello_world_linux)}"
hello world linux
```

以上用 _ 分隔这段字符串。

请注意，您可以使用 '.' 或 ':' 作为分隔符。

示例：

- (s.:.): 分割字符串是 `:`
- (s:.:)：分割字符串是 `.`

如下为译者补充示例：

```bash
echo "${(s.:.)$(echo hello:world.linux)}"
hello world.linux
$ echo "${(s:.:)$(echo hello:world.linux)}"
hello:world linux
```

## 总结

从本文可知，zsh 提供了相当多的能力，初看可能略感复杂。但如果你坚持使用，它将会成为你日常工作中不可获取的工具。

我的博文：[从 0 开始：教你如何配置 zsh](http://localhost:1313/posts/2023-09-16-how-to-use-zsh-a-beginner-guide/),译：[How to Use Zsh: A Beginer's Guide](https://linuxopsys.com/topics/use-zsh)
