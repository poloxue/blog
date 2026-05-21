---
title: "ripgrep - 更高效的文件搜索"
weight: 2
---

# ripgrep

[ripgrep](https://github.com/BurntSushi/ripgrep) 是一款文本搜索命令，功能与 grep 类似。和 fd 之于 find 一样，ripgrep 在体验和性能上同样完胜 grep。

## 安装

```zsh
brew install ripgrep
```

## 递归

ripgrep 的默认行为也是递归搜索，命令为 rg pattern，且默认高亮显示，与 `grep --color main . -nR` 对比，明显更加简洁易用，体验更好。

示例效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-09.gif)

## 指定目录

ripgrep 最后的参数即可指定搜索的目录。

```zsh
rg main ~/Code/golang-examples/
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-10.gif)

## 指定文件

搜索指定文件与指定目录类似，命令的最后一个参数指定即可。

```zsh
rg main ~/Code/golang-examples/main.go
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-11.png)

## 通配符

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

## 正则

ripgrep 支持通过 `-e` 选项启用正则表达式搜索，如搜索文件中的指定日期格式内容。

```zsh
rg -e '[0-9]{2}:[0-9]{2}'
```

## 默认过滤

和 fd 一样，ripgrep 的高效率搜索能力一方面也是因为默认忽略了一些文件，如它忽略隐藏文件以及 .gitgnore .ignore .rgignore 中的文件。禁用 ignore 可通过选项 --no-ignore 即可。

对于隐藏文件，通过 `--hidden` 即可搜索包含隐藏文件。

如果希望彻底禁用隐藏能力，通过 `-uuu` 实现，如 `rg -uuu pattern`。

## 大小写敏感

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

## 搜索并替换

ripgrep 中，我们通过 `-r` 可直接替换搜索结果，但不改变原内容。

```zsh
rg main ~/Code/golang-examples r main # 只替换输出，未修改文件
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-12.png)

## 配置

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

