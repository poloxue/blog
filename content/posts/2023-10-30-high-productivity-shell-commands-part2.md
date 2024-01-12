---
title: "我的终端环境：高效 shell 命令（二）之查找搜索 - fd ripgrep fzf"
date: 2023-10-30T20:13:53+08:00
draft: false
comment: true
description: "本文将介绍一些高效的查找搜索命令，分别是 fd、ripgrep 与 fzf，提升命令内容的查找效率"
tags: ["zsh"]
---

{{< video bb_id="962959372" yt_id="CcnmD_dHsnk" >}}
本文将介绍一些用于高效查找和搜索的命令，它们分别是 fd、ripgrep 与 fzf。相对于常用的 grep 和 find，这些命令在性能有大部分提升，而且，它们的使用方式上更符合人性。

如下是这几个命令的简单说明：

- [fd](https://github.com/sharkdp/fd)，目录与文件搜索命令，比默认 find 更易于使用，而且查找速度上更快；
- [ripgrep](https://github.com/BurntSushi/ripgrep)，用于高效的内搜索容；
- [fzf](https://github.com/junegunn/fzf)，是一款命令行交互式的模糊搜索工具，可与其他命令进行结合，提高使用体验；

其中的 fd 与 ripgrep 是用 rust 编写，性能上完虐传统的 find 与 grep。

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

## [fd](https://github.com/sharkdp/fd)

fd 是一款文件查找的命令，可用于替换默认的 find，它在体验上更加友好，且查询效率极高。

### 安装

```zsh
brew install fd
```

### 使用

案例一 简单搜索 - fd pattern

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-01.gif)

案例二 正则查询 - fd regex_exp

如查找包含文件名中包含日期的文件：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-02.gif)

或者查找所有的 go 代码文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-03.gif)

这个表达式更加正确的表述是 fd `.*\.go$`，查出所有以 `.go` 结尾的文件。

案例三 通配符

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-04.gif)

案例四 指定类型

前面的查找 go 文件的方式，无论是通过正则还是通配符，都相对繁琐一点，可直接通过 fd -e go pattern 的模式直接寻找指定扩展的文件。

如下示例查找以 `.py` 为扩展且文件名包含 `moving_average` 的文件：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-05.gif)

如果无 patern，效果则是查找所有的 `.py` 的文件，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-07.gif)

案例五 隐藏文件和 `.gitinogre`

默认情况下，fd 的查找结果不包含隐藏文件，可通过 -H 选项启用：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-06.png)

从上图可以看出，其中包含了不少 .git/config 文件，而 .git 是隐藏文件。

查询范围包含 gitignore

同样，除了默认忽略 hidden 文件，fd 还会忽略 .gitignore 中的文件，选项 -I 即可查找包含在 `.gitinogre/.fdignore/.ignore` 的文件，形如 `fd -I pattern`。

其他示例

案例                    | 命令                          | 说明
----------------------- | ----------------------------- | -------
忽略大小写              | fd -i readme.md               | 包含 readme.md 和 README.md
大小写的 smartcase 模式 | fd -s readme.md               | 包含 readme.md 和 README.md
---                     | fd -s README.md               | 只包含 README.md
查找结果显示详细信息    | fd -l .go                     | 查询结果显示文件详情
大小过滤                | fd -S +1000k                  | 查询大小 > 1000k 的文件
仅查找目录              | fd --type/-t directory golang | 查询所有目录文件
查询并执行命令          | fd --type/-t file -x wc -l    | 查询文件并计算行数
---                     | fd --type/-t file -X vim      | 查询所有文件并用 vim 打开

### 性能

通过 hyperfine 或 time 命令测试，与 find 进行性能对比，包含所有文件且大小写不敏感搜索。

```zsh
hyperfine --warmup 3 "find . -iname *.go" "fd -i -g '*.go' -uu"
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-08.gif)

## [ripgrep](https://github.com/BurntSushi/ripgrep)

ripgrep 是一款文本搜索命令，功能与 grep 类似；

### 安装

```zsh
brew install ripgrep
```

### 使用：

用法一 默认递归搜索当前目录，如 rg pattern；

默认高亮显示，对比于 `grep --color main . -nR` 简洁易用，体验更好。效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-09.gif)

用法二 搜索指定目录，rg patern directory

```zsh
rg main ~/Code/golang-examples/
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-10.gif)

用法三 搜索指定文件，rg pattern filepath

```zsh
rg main ~/Code/golang-examples/main.go
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-11.gif)

用法四 通配符禁用目录

通过通配符方式，禁用目录递归。

```zsh
rg main -g '!/*/*'
```

其他还可实现如排除文件：

```zsh
rg main -g '!main.go'
```

排除目录

```zsh
rg -g '!directory'
```

用法五 正则搜索

通过 `-e` 选项启用正则表达式搜索，如： 

```zsh
rg -e '[0-9]{2}:[0-9]{2}'
```

用法六 默认过滤

和 fd 一样，ripgrep 的高效率搜索能力一方面也是因为默认忽略了一些文件，如它忽略隐藏文件以及 .gitgnore .ignore .rgignore 中的文件。禁用 ignore 可通过选项 --no-ignore 金融即可。而对于隐藏文件，通过 `--hidden` 即可搜索包含隐藏文件。

如果希望禁用所有隐藏文件，则可通过 `-uuu` 实现，如 `rg -uuu pattern`。

用法八 大小写敏感：

默认大小写敏感：rg pattern，可通过 `-i` 选项忽略大小写：`rg -i pattern`，或者通过 `-S` 启用 smartcase 模式，即 `rg -S pattern`。

用法九 搜索并替换

通过 `-r` 可直接替换搜索结果，但不改变原内容。

```zsh
rg main ~/Code/golang-examples r main # 只替换输出，未修改文件
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-12.png)

### 配置

配置文件修改 ripgrep 的默认行为，通过设置环境变量 `RIPGREP_CONFIG_PATH` 执行配置文件。配置文件中的设置方式，和上文介绍的 bat 类似，都是通过 `--xxx` 选项的形式设置。

```zsh
# Don't let ripgrep vomit really long lines to my terminal, and show a preview.
--max-columns=150
--max-columns-preview

# Add my 'web' type.
--type-add
web:*.{html,css,js}*

# Search hidden files / directories (e.g. dotfiles) by default
--hidden

# Using glob patterns to include/exclude files or folders
--glob=!.git/*

# or
--glob
!.git/*

# Set the colors.
--colors=line:none
--colors=line:style:bold

# Because who cares about case!?
--smart-case
```

如希望 ripgrep 默认 smartcase 能力，可将 `--smart-case` 直接配置到配置文件中。

## [fzf](https://github.com/junegunn/fzf) 

fzf 全名 fuzzy finder，是一款通用的命令行模糊查找工具。它可与其他命令结合，提升其他命令的使用体验。

### 安装

```zsh
brew install fzf
```

### 使用

用法一 目录搜索

默认在当前所在目录进入交互式目录文件搜索路径，效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-13.gif)

支持通过 CTRL+P/N 上下选择，确认搜索结果后，输入 Enter 确认后，它会将输出直接作为输出打印到标准输出。

用法二 将其他命令输出作为 fzf 的搜索内容；

除了默认基于当前位置搜索文件目录，fzf 的搜索内容可自定义。如将 `one`、`two`、`three`、`four` 作为输入，通过 fzf 搜索选择。

```zsh
echo "one\ntwo\nthree\nfour" | fzf
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-14.gif)

如此一来，更过可能就诞生了，如 ls 的文件作为输入：

```zsh
ls | fzf
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-15.gif)

将上面介绍的 fd 的查找结果作为输入：

```
fd --type file | fzf
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-16.gif)

用法三 将搜索结果作为其他命令的输入；

fzf 的搜索结果如果只是打印到终端，那就太可惜了，可将其其它命令的输入。如下命令：

```zsh
vim `fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-17.gif)

将 fzf 的搜索结果作为 vim 命令的输入。

用法四 输入输出，命令组合实现更多可能

实验到这里，是不是已经发现，fzf 实现更多可能性的能力，形如 `command1 | fzf | xargs command2`，将 command1 的结果作为 fzf 的搜索来源，将 fzf 的确认结果作为 command2 的输入。

如上节的 zoxide 快速进入文件能力。

```
cd `zoxide query --list {querystring} | fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-19.gif)

默认的 `zoxide query --list` 只记录的是 zoxide 中的历史记录，如想实现全局查询，可替换为类似如下命令：

```zsh
cd `fd --type=directory | fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-18.gif)

如下是一个 cdg 的命令实现，可实现 cd 命令的任意目录的快速交互式搜索：

```zsh
alias cdg='cd_global() {cd $(fd --type directory $1 $2 | fzf)}; cd_global'
```

使用方式形如 `cdg pattern directory`，在哪一个目录下查询包含 pattern 的目录，确认后即可 cd 进入到这个目录。


![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-20.gif)

说明下，我实际使用下来，这个命令确实很少需要，因为大多数时间我还是在访问我经常使用的目录，默认 zoxide 的能力完全足够使用。

## 总结

本文介绍了三个与查找搜索相关的命令，分别是用于文件查找的 fd、文本搜索的 ripgrep 及交互式搜索工具 fzf。通过将这几个命令的组合使用，再进一步提升我们的终端使用效率。

