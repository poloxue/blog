---
title: "推荐 3 个高效搜索命令"
date: 2023-10-30T20:13:53+08:00
draft: false
comment: true
description: "本文将介绍一些高效的查找搜索命令，分别是 fd、ripgrep 与 fzf，提升命令内容的查找效率"
tags: ["zsh"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-00.png)

本文将介绍三个高效搜索命令，分别是 fd、ripgrep 与 fzf。

我将介绍的 fd 和 ripgreap 对标的是传统 grep 和 find，它们在性能和使用体验上都有大幅提升。如果你成为 10x 程序员，强烈推荐使用它们。

**[fd](https://github.com/sharkdp/fd)**，目录与文件搜索命令，比默认 find 更易于使用，而且查找速度上更快；

**[ripgrep](https://github.com/BurntSushi/ripgrep)**，可用于高效的内容搜索，比默认的 grep 命令速度更快；

**[fzf](https://github.com/junegunn/fzf)**，命令行交互式模糊搜索工具，可与其他命令进行结合，提高使用体验；

如上的 fd 与 ripgrep 是由 rust 编写，性能上完虐传统的 find 与 grep。

视频版本，没有文章详细。

{{< video bb_id="962959372" yt_id="CcnmD_dHsnk" >}}

## [fd](https://github.com/sharkdp/fd)

fd 是一款文件查找命令，可替换系统默认 find，它的体验更友好，且查询效率极高。我们在使用传统的 find 时，要经常查手册看帮助文档，但使用 fd，它的默认行为就能满足我们大部分的需求。

如何使用呢？

### 安装

```zsh
brew install fd
```

### 递归

文件系统下搜索文件名，最常见的场景是递归搜索文件名包含 pattern 的文件，不知道有你是否能立刻想起来 find 如何写呢？

示例：遍历查找。

```bash
$ fd pattern
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-01.gif)

### 正则

如果要正则查询，pattern 默认即支持正则表达式。

查找文件名包含日期的文件：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-02.gif)

或者查找所有的 go 代码文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-03.gif)

这个表达式更加正确的表述是 fd `.*\.go$`，查出所有以 `.go` 结尾的文件。

### 通配符

fd 同样是支持统配符的，通过 `-g` 选项指定通配符。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-04.gif)

### 文件类型

前面的查找 go 文件，其实也是查找类型是 Go 源码。但正则和通配符，都比较繁琐，可直接通过 `fd -e go pattern` 的模式直接寻找指定扩展的文件。

示例：查找 python 源码文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-05.gif)

如果无需 patern，则表明查找所有的 `.py` 的文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-07.gif)

### 隐藏文件和 `.gitinogre`

fd 搜索效率高的一个原因是它默认不查找隐藏文件和 `gitignore` 文件。

如想启用隐藏文件的查找，可指定 -H 选项，示例如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-06.png)

从上图可知，其中包含了隐藏目录 .git 下的文件。

除了默认忽略 hidden 文件，fd 还会忽略 .gitignore 中的文件，选项 -I 即可查找包含在 `.gitinogre/.fdignore/.ignore` 的文件。

```bash
$ fd -I pattern
```

### 其他示例

fd 的更多选项，我就不一一介绍了，可以查看如下表格。

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

除了忽略隐藏和 gitignore 中的文件外，fd 在性能上也是碾压 find。通过 hyperfine 命令测试 fd 与 find。

```zsh
hyperfine --warmup 3 "find . -iname *.go" "fd -i -g '*.go' -uu"
```

性能对比结果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-08.gif)

压测结果显示，fd 相较于 find 快了 3.88 倍。

## [ripgrep](https://github.com/BurntSushi/ripgrep)

ripgrep 是一款文本搜索命令，功能与 grep 类似。和 fd 之于 find 一样，ripgrep 在体验和性能上同样完胜 grep。

### 安装

```zsh
brew install ripgrep
```

### 递归

ripgrep 的默认行为也是递归搜索，命令为 rg pattern，且默认高亮显示，与 `grep --color main . -nR` 对比，明显更加简洁易用，体验更好。

示例效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-09.gif)

### 指定目录

ripgrep 最后的参数即可指定搜索的目录。

```zsh
rg main ~/Code/golang-examples/
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-10.gif)

### 指定文件

搜索指定文件与指定目录类似，命令的最后一个参数指定即可。

```zsh
rg main ~/Code/golang-examples/main.go
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-11.png)

### 通配符

我们使用 `-g` 通过通配符指定搜索路径。

如下是禁用目录递归：

```zsh
rg main -g '!/*/*'
```

还可实现如排除指定的文件：

```zsh
rg main -g '!main.go'
```

或者排除指定的目录

```zsh
rg -g '!directory'
```

### 正则

ripgrep 支持通过 `-e` 选项启用正则表达式搜索，如搜索文件中的指定日期格式内容。

```zsh
rg -e '[0-9]{2}:[0-9]{2}'
```

### 默认过滤

和 fd 一样，ripgrep 的高效率搜索能力一方面也是因为默认忽略了一些文件，如它忽略隐藏文件以及 .gitgnore .ignore .rgignore 中的文件。禁用 ignore 可通过选项 --no-ignore 即可。

对于隐藏文件，通过 `--hidden` 即可搜索包含隐藏文件。

如果希望彻底禁用隐藏能力，通过 `-uuu` 实现，如 `rg -uuu pattern`。

### 大小写敏感

ripgrep 默认即大小写敏感。

```bash
rg pattern
```

通过 `-i` 选项可忽略大小写。

```bash
rg -i pattern
```

或者通过 `-S` 启用 smartcase 模式，即

```bash
rg -S pattern
```

### 搜索并替换

ripgrep 中，我们通过 `-r` 可直接替换搜索结果，但不改变原内容。

```zsh
rg main ~/Code/golang-examples r main # 只替换输出，未修改文件
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-12.png)

### 配置

ripgrep 支撑配置文件修改 ripgrep 的默认行为。我们通过设置环境变量 `RIPGREP_CONFIG_PATH` 指定配置文件路径。

配置文件的配置方式和上文介绍的 bat 类似，都是通过 `--xxx` 选项的形式设置。

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

如我们希望 ripgrep 默认启用 smartcase 能力，可将 `--smart-case` 直接配置到配置文件中。

## [fzf](https://github.com/junegunn/fzf) 

fzf 全名 fuzzy finder，一款通用的命令行模糊查找工具。它可与其他命令结合，提升其他命令的使用体验。

### 安装

```zsh
brew install fzf
```

### 目录搜索

fzf 的默认行为是在当前目录搜索。当然它的搜索和 fd 的搜索不同，它会进入交互式模式搜索文件或目录路径。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-13.gif)

它支持通过 CTRL+P/N 上下选择，确认搜索结果后，输入 Enter 确认后，它会将输出直接作为输出打印到标准输出。由于它输出为标准输出，我们就有了更多可能性，通过管道将它与其他命令结合。

### 命令组合

fzf 最大的魅力在于，我们可将其与其他命令组合，如

- 将其他命令的输出作为 fzf 的输入，基于它进行搜索；
- 或将 fzf 的搜索内容作为其他命令的输入，更智能使用其他命令。

我们具体介绍下吧。

首先，我们示例将 `one`、`two`、`three`、`four` 作为输入，通过 fzf 搜索选择。

```zsh
echo "one\ntwo\nthree\nfour" | fzf
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-14.gif)

更多可能就诞生了，我们可以讲 ls 的输出作为 fzf 的输入。

```zsh
ls | fzf
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-15.gif)

将 fd 查找结果作为 fzf 的输入。

```bash
fd --type file | fzf
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-16.gif)

接下来，我们尝试将搜索结果作为其他命令的输入。毕竟，如果 fzf 的搜索结果只是输出到终端，那就太可惜了，可将其作为其它命令的输入。

如将 fzf 搜索结果作为 vim 的输入，助力 vim 快速打开文件。

```zsh
vim `fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-17.gif)

我们也可以将多个命令组合，fzf 从其他命令接收输入，同时将搜索结果输出给其他啊命令。这样就能实现更多可能。

是不是已经发现，fzf 实现更多可能性的能力，如 `command1 | fzf | xargs command2`，将 command1 的结果作为 fzf 的搜索来源，将 fzf 的确认结果作为 command2 的输入。

如上文中的 zoxide （高效 cdi 命令）快速进入文件能力，我们可用 fzf 实现：

```
cd `zoxide query --list {querystring} | fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-19.gif)

默认的 `zoxide query --list` 只记录的是 zoxide 中的历史记录，如想实现进入任意的目录，可使用如下命令：

```zsh
cd `fd --type=directory | fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-18.gif)

如下是一个 cdg 命令，作用是交互式搜索全局任意目录并通过 cd 进入。可 zoxide 中的 cdi 对比。

```zsh
alias cdg='cd_global() {cd $(fd --type directory $1 $2 | fzf)}; cd_global'
```

使用方式形如 `cdg pattern directory`，在哪一个目录下查询包含 pattern 的目录，确认后即可 cd 进入到这个目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-20.gif)

## 总结

本文介绍了三个实现终端高效搜索的命令，分别是用于文件查找的 fd、文本搜索的 ripgrep 及交互式搜索工具 fzf。希望通过将这几个命令的组合使用，再进一步提升我们的终端使用效率。

希望本文对你有所帮助，感谢阅读。
