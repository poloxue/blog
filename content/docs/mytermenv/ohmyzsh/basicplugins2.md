---
date: 2024-03-03
title: "基础插件 2"
weight: 3
---

# 基础插件 2

本文是基础篇插件的第二篇，继续介绍 4 个常用的 oh-my-zsh 插件。

| 插件 | 作用 |
|------|------|
| `copypath` | 拷贝当前目录路径到剪贴板 |
| `copyfile` | 拷贝文件内容到剪贴板 |
| `dirhistory` | 用方向键在目录历史中导航 |
| `history` | 更好的历史命令查看方式 |

在 `~/.zshrc` 中加上：

```bash
plugins=(... copypath copyfile dirhistory history)
```

## copypath

有时候需要把当前目录路径粘到某个对话框或文档里。`copypath` 插件提供了一个 `copypath` 命令，一键把当前工作目录的绝对路径复制到剪贴板。

```bash
cd ~/Projects/myapp/src
copypath    # 剪贴板已复制: /Users/poloxue/Projects/myapp/src
```

搭配 Finder 的"前往文件夹"（Cmd+Shift+G）使用，粘贴路径直达目标目录。

## copyfile

当你需要把某个文件的内容复制到剪贴板时，不用 `cat file | pbcopy` 或鼠标选中，直接用 `copyfile`：

```bash
copyfile main.go   # main.go 的内容已复制到剪贴板
```

在粘贴代码到邮件、文档或聊天工具时非常好用。

## dirhistory

这个插件让你用方向键在目录访问历史中导航：

| 快捷键 | 动作 |
|--------|------|
| `Alt + ←` | 回到上一个访问的目录 |
| `Alt + →` | 前进到下一个目录 |
| `Alt + ↑` | 回到父目录 |

在几个目录之间来回切换时非常顺手，不用反复 `cd ..` 再 `cd project`。

## history

oh-my-zsh 默认已经增强了历史命令搜索（输入一部分然后按方向键上），但 `history` 插件提供了更方便的历史查看：

```bash
# 查看最近的 10 条历史
history 10

# 搜索包含特定关键字的命令
history | grep docker

# 查看今天执行的命令
history -t today
```

## 小结

这 4 个插件专注于剪贴板和导航增强，虽然不是必装，但用上了就能感受到细节上的便利。下一篇开始介绍更强大的社区插件。
