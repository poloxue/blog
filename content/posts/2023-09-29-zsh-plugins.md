---
title: "ZSH 常用插件安装与使用"
date: "2023-09-28T10:00:00"
draft: true
comment: true
tags: ["zsh"]
---

上篇文章 [从 0 开始：教你如何配置 zsh](https://www.poloxue.com/posts/2023-09-16-how-to-use-zsh-a-beginner-guide/) 是一篇译文，介绍了 zsh 的使用基于，还有如何通过 oh-my-zsh 安装插件。其中，已经介绍了一些插件的安装与使用。

本篇文章，将推荐几个我常用的插件或命令。

## Git

首先，在 oh-my-zsh 安装成功后，默认开启的插件 `git` 插件。它提供了一些与 Git 相关命令的别名和函数。

如下所示的一些常用命令：

```bash
ga  -> git add
gc  -> git commit --verbose
gaa -> git add --all
gb  -> git branch
gp  -> git push
gl  -> git pull
...
```

这里只列举了部分，更多信息，推荐阅读插件的 [README.md 文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)

如果不习惯于 Git 的图形界面工具，就需要时常与 Git 的终端命令打交道。利用 git 插件提供的缩写，这将对我们的操作效率有很大提升。

## z

z 插件，这个命令的名字挺简洁的，但它确实是个神器。

## 语法高亮

语法高亮插件将使用 [zsh-highlight-syntax](https://github.com/zsh-users/zsh-syntax-highlighting) 插件。

这个插件的主要作用是提供 zsh 的语法高亮。

