---
title: "什么是 Shell"
date: 2023-09-19T14:06:02+08:00
comment: true
draft: false
tags: ["shell", "zsh"]
---

## 何为 Shell?

简言之，shell 是一个键盘命令和操作系统间的搬运工，将用户输入交给系统执行的程序。曾几何时，它是 Linux 等类 Unix 系统上唯一的操作方式。如今，除了 Shell 等命令行界面 (CLI) ，我们还有了图形用户界面 (GUI)。

bash（全称 Bourne Again Shell，最初由 Steve Bourne 开发，它是 sh，初版 Unix Shell 的增强版本，是绝大多数 unix-like 系统的默认 shell。其比较著名的 Shell 还有 ksh、tcsh 和 zsh。

## 何为 Terminal？

Terminal，"终端"，一个称为 Terminal Emulator（"终端模拟器"）的程序，通过窗口实现与 Shell 交互。常见的 Linux 发行版支持的 Terminal 有 gnome-terminal、konsole、xterm、rxvt、konsole、nxterm、eterm 等。

## 启动终端

Window Manager（窗口管理器），通常会支持从菜单启动终端。

请通过它打开你的程序列表，找到一个看起来像终端的图标。无论什么终端，它的核心能力一定是访问 shell。当然，每个 terminal 会有自己的特色，选择一款你偏爱的。

## 键盘测试

让我们尝试在终端输入一些字符。

首先引入眼帘的是 shell 提示符，多数情况下，提示符的组成包含用户名和机器名称，并以 $ 结尾，效果如下：

```bash
[me@linuxbox me]$
```

赞！让我们输入一些无用字符，

如下所示：

```bash
[me@linuxbox me]$ kdkjflajfks
```

输入 "Enter" 确认。我们将收到一条 "error message"。

如下所示：

```bash
[me@linuxbox me]$ kdkjflajfks
bash: kdkjflajfks: command not found
```

输入 "↑" 键。终端将会显示之前执行的命令 "kdkjflajfks"。这是命令历史记录。输入 "↓" 键，我们会再次回到空行。

输入 "↑" 回到 “kdkjflajfks” 命令。使用 "←" 键和 "→" 键将光标放置在命令行的任何位置，删除输入进行错误纠正。

> 注：如果 shell 输入提示符以 `#` 而非 `$` 结尾，这表明现在是 super user 身份，你拥有超级管理权限。这也意味着，你有权删除或覆盖系统中的任何文件。这很危险，应尽量禁止，除非你有绝对需求。

## 使用鼠标

Shell 虽然是为命令行而设计的，但适当的使用鼠标，会带给你更好的体验。如使用鼠标进行滚动与复制粘贴，能进行快速的定位与拷贝。

## 窗口聚焦...

当安装 Linux 和其 Window Manager（如 Gnome 或 KDE）时，默认配置在某些方面的行为类似旧版 OS。

例如，它的 focus 策略可能是 “单击焦点”。要使窗口 focus（变为 Active），就要在窗口中单击。这与传统的 X Window 不同。你可以考虑将其设置为“焦点跟随鼠标”。

期初，你或许不太适应，窗口在获得焦点时不会被推到前面（单击窗口才能执行此操作），但能够处理多个窗口的同时，当前的活动窗口还不会遮挡了另一个窗口。

不防尝试一下，或许你会喜欢这个策略。我们在窗口管理器的配置中可以它的设置。

译：[What is "the Sell"?](https://linuxcommand.org/lc3_lts0010.php)

