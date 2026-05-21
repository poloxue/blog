---
title: "为什么选择 zsh ？"
weight: 2
---

# 为什么选择 zsh ？

开始前，先问为什么，知其然，要知其所以然，是个好习惯。所以，为什么要用 zsh 呢？

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


