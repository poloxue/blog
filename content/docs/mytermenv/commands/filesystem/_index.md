---
date: 2024-03-24
description: "文件目录管理命令"
title: "文件目录"
weight: 1
hideBreadcrumb: true
---


# 文件目录

我将先介绍平时工作中最常用的与目录文件相关的命令，分别是替换 ls 的 [exa](https://github.com/ogham/exa)，替换 cd 的 [zoxide](https://github.com/ajeetdsouza/zoxide) 和替换 cat 的 [bat](https://github.com/sharkdp/bat)。它们的优势会在文章中逐步展开说明。

特别提醒：exa 已停止维护，可用 exa 的 fork 版本 [eza](https://github.com/eza-community/eza) 替代。

---

## bat

# bat

说完了 ls 列举目录，cd 进入目录，我们继续介绍一个命令，[bat](https://github.com/sharkdp/bat) 查看文件内容。

这个 bat 和 Baidu/Alibaba/Tencent 没有联系，它是一款支持语法高亮、GIT 集成的用于替换类 Unix 系统下快速查看文件内容的命令，功能与 cat 相似的命令。

---

## exa

# exa 

首先是 exa，一款可用于替换系统默认 ls 的命令，在平时工作中 ls 几乎使用最多的命令，而 exa 在支持 ls 的基本能力基础上，提供了更丰富的特性。

## 快速安装

---

## zoxide

# zoxide

在正式介绍 [zoxide](https://github.com/ajeetdsouza/zoxide) 前，尝试提前问自己一个问题，Linux 默认命令 cd 好不好用？我的答案是，相当难用，无论多么丝滑的操作，一旦遇到 cd，只能说一句 f**k。

你是不是经常这样使用 cd 呢？