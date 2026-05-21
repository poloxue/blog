---
title: "bat"
weight: 3
---

# bat

说完了 ls 列举目录，cd 进入目录，我们继续介绍一个命令，[bat](https://github.com/sharkdp/bat) 查看文件内容。

这个 bat 和 Baidu/Alibaba/Tencent 没有联系，它是一款支持语法高亮、GIT 集成的用于替换类 Unix 系统下快速查看文件内容的命令，功能与 cat 相似的命令。

我们直接介绍它的安装与使用吧。

### 安装

```zsh
brew install bat # 其他系统请查看 GithHub README.md
```

### 使用

对于 bat 命令，我先介绍它的使用，然后再谈配置，因为配置并非它的必选项而是优化项。

bat 相比于 cat 的第一个优势，就是它支持语法高亮效果与行号显示。如我们查看一个 Go 的源码文件，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-08.gif)

而且，bat 还集成 Git。如下我们修改了 logger.go 文件，通过 bat 即可查看它的修改点；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-09.gif)

默认情况下，bat 采用分页输出，这对于读取大文件非常有帮助，不用担心失误导致产生一大片控制台输出。但如果你希望 bat 和 cat 一样，一次性无分页输出文本，可通过 `--pager=never` 或 `--no-pager` 选项实现。

```zsh
bat --pager=never logger.go
bat --no-pager logger.go
```

如果你习惯使用 cat 的模式，希望默认不启用分页能力，可直接在配置文件配置默认行为，在其中增加 `--pager=never`。

接下来说说如何通过 bat 的配置改变它的默认行为吧。

### 配置


bat 的配置文件路径是通过环境变量指定的。我们在 `.zshrc` 中设置 bat 配置文件位置环境变量。

```bash
export BAT_CONFIG_PATH="${XDG_CONFIG_HOME:-~/.config/bat.conf"
```

生效后，执行如下命令将会生成配置文件：

```zsh
bat --generate-config-file
```

生成配置文件，位于 `~/.config/bat.conf`。

假设我不喜欢 bat 默认的主题，就可以通过配置修改了。如配置 bat 默认选项，将主题改为 `--theme=TwoDark` 启用：


```zsh
# Specify desired highlighting theme (e.g. "TwoDark"). Run `bat --list-themes`
# for a list of all available themes
--theme=TwoDark
```

如果你想查看更多主题，可通过 `bat --list-themes` 查看 bat 支持的主题列表。

现在，不想启用 bat 的分页能力，在配置中添加：

```bash
--pager=never
```

### 别名

觉得 bat 不错，想直接替换 cat 命令，在 `zshrc` 中配置别名即可，将默认 cat 命令，替换为 bat，如下所示：

```bash
alias cat='bat'
```
