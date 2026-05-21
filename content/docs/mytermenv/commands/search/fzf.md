---
title: "fzf - 交互式命令行查找器"
weight: 3
---

# fzf - 交互式查找器

[fzf](https://github.com/junegunn/fzf) 全名 fuzzy finder，一款通用的命令行模糊查找工具。它可与其他命令结合，提升其他命令的使用体验。

## 安装

```zsh
brew install fzf
```

## 目录搜索

fzf 的默认行为是在当前目录搜索。当然它的搜索和 fd 的搜索不同，它会进入交互式模式搜索文件或目录路径。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-13.gif)

它支持通过 CTRL+P/N 上下选择，确认搜索结果后，输入 Enter 确认后，它会将输出直接作为输出打印到标准输出。由于它输出为标准输出，我们就有了更多可能性，通过管道将它与其他命令结合。

## 命令组合

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

## 实际场景

是不是已经发现，fzf 实现更多可能性的能力，如 `command1 | fzf | xargs command2`，将 command1 的结果作为 fzf 的搜索来源，将 fzf 的确认结果作为 command2 的输入。

如 cd 的替代命令 zoxide，它有个高效 cdi 命令，能快速进入某个目录，我们可用 fzf 实现：

```bash
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
