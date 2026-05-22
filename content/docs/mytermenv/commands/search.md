---
date: 2024-04-14
title: "搜索查找"
weight: 2
description: ""
---

# 搜索查找

本文将介绍三个高效搜索命令，分别是 fd、ripgrep 与 fzf。fd 和 ripgrep 对标的是传统 grep 和 find，它们在性能和体验上有大幅提升。如果你想成为 10x 程序员，强烈推荐使用它们。

---

## fd

[fd](https://github.com/sharkdp/fd) 是一款文件查找命令，可替换系统默认 find。它的体验更友好，且查询效率极高。使用传统的 find 时，经常要查手册看帮助文档，但使用 fd，它的默认行为就能满足我们大部分的需求。

### 安装

```bash
# macOS
brew install fd

# Debian/Ubuntu
sudo apt install fd-find
```

### 基础使用

与 find 对比一下就知道了：

```bash
# find 查找 Python 文件
find . -name "*.py" -type f

# fd 查找 Python 文件（默认递归，无需 -name -type）
fd "*.py"

# find 搜索包含 "main" 的文件
find . -type f -name "*main*"

# fd 搜索包含 "main" 的文件
fd main
```

fd 的默认行为比 find 聪明得多：

- 自动递归搜索，不用像 find 那样手动 `-type f`
- 默认忽略隐藏文件和 `.gitignore` 中的文件（搜项目代码时不会带出 node_modules）
- 彩色高亮匹配内容

### 正则搜索

```bash
fd "^config\."     # 搜索以 config. 开头的文件
fd "[0-9]{4}"      # 搜索包含 4 位数字的文件
```

### 指定搜索路径

```bash
fd pattern ~/Projects    # 在指定目录下搜索
fd -H pattern ~          # 包含隐藏文件
```

### 搭配命令执行

fd 真正的威力在于配合 `-x` 或 `-X` 执行命令：

```bash
# 查找所有 Go 文件并统计行数
fd -e go -x wc -l

# 查找并删除所有 .tmp 文件
fd -e tmp -X rm

# 查找并替换文件中的内容
fd -e md -x sed -i 's/old/new/g'
```

`-x` 对每个匹配的文件分别执行，`-X` 一次性传入所有匹配的文件。

### 文件类型过滤

```bash
fd -e py          # 只搜索 Python 文件
fd -e md -e txt   # 搜索 Markdown 和文本文件
fd -t f           # 只搜索文件
fd -t d           # 只搜索目录
fd -t l           # 只搜索符号链接
```

### 性能

fd 使用 Rust 编写，通过并行遍历文件系统和智能跳过忽略文件，搜索速度比 find 快 5-10 倍。

---

## ripgrep

[ripgrep](https://github.com/BurntSushi/ripgrep) 是一款文本搜索命令，功能与 grep 类似，在体验和性能上完胜 grep。

### 安装

```bash
brew install ripgrep
```

### 基础使用

ripgrep 默认递归搜索，命令为 `rg pattern`，且默认高亮显示，比 `grep --color main . -nR` 简洁得多：

```bash
rg main ~/Code/golang-examples/
```

### 通配符

使用 `-g` 通过通配符指定搜索路径：

```bash
rg main -g '!main.go'    # 排除指定文件
rg -g '!directory'       # 排除指定目录
rg -g '!/*/*'             # 禁用目录递归
```

### 正则

```bash
rg -e '[0-9]{2}:[0-9]{2}'   # 搜索日期格式
```

### 默认过滤

和 fd 一样，ripgrep 高性能的原因之一也是默认忽略隐藏文件和 `.gitignore`、`.rgignore` 中的文件。禁用忽略用 `--no-ignore`。搜索隐藏文件用 `--hidden`。彻底禁用过滤用 `-uuu`。

### 大小写

```bash
rg pattern        # 默认大小写敏感
rg -i pattern     # 忽略大小写
rg -S pattern     # smartcase 模式
```

### 搜索并替换

```bash
rg main -r new_content    # 替换搜索结果，不修改原文件
```

### 配置

通过环境变量 `RIPGREP_CONFIG_PATH` 指定配置文件：

```bash
--max-columns=150
--max-columns-preview
--smart-case
--hidden
--glob=!.git/*
```

---

## fzf

[fzf](https://github.com/junegunn/fzf) 全名 fuzzy finder，是一款通用的命令行模糊查找工具。它可以与其他命令结合，堪称终端管道中的瑞士军刀。

### 安装

```bash
# macOS
brew install fzf

# 安装额外的小工具（Ctrl+T、Alt+C 等绑定）
$(brew --prefix)/opt/fzf/install
```

### 核心概念

fzf 的核心是一个交互式模糊搜索界面：

```bash
cat /usr/share/dict/words | fzf
```

输入几个字母，fzf 会模糊匹配所有包含这些字母的行，按匹配度排序并实时高亮。

### 文件搜索

fzf 与 fd 结合，效果爆炸：

```bash
# 交互式搜索文件，选中后用 nvim 打开
fd --type f | fzf | xargs nvim

# 或用 Ctrl+T（需要 source 补全脚本）
```

### 目录跳转

```bash
# 一行命令实现交互式 cd
cd $(fd --type d | fzf)

# 封装成函数
function fcd() {
  local dir
  dir=$(fd --type d --hidden --follow --exclude .git | fzf) && cd "$dir"
}
```

### 历史命令搜索

在 `~/.zshrc` 中加一行，替换默认的 `Ctrl+R`：

```bash
source <(fzf --zsh)
```

之后按 `Ctrl+R` 弹出交互式搜索框，输入关键字模糊搜索历史命令，选中后回车执行。比 bash 默认翻页式搜索直观得多。

### 进程搜索

```bash
kill -9 $(ps aux | fzf | awk '{print $2}')
```

### Git 集成

```bash
# 交互式切换分支
git checkout $(git branch | fzf | tr -d ' *')

# 交互式查看 git 日志（带预览）
git log --oneline --graph | fzf --preview 'git show {1}'
```

`--preview` 是 fzf 的强力功能——选中时自动预览选中项的内容，不用打开额外窗口。
