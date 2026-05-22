---
date: 2024-03-24
title: "文件目录"
weight: 1
description: ""
---


我将先介绍平时工作中最常用的与目录文件相关的命令，分别是替换 ls 的 [eza](https://github.com/eza-community/eza)，替换 cd 的 [zoxide](https://github.com/ajeetdsouza/zoxide) 和替换 cat 的 [bat](https://github.com/sharkdp/bat)。它们的优势会在文章中逐步展开说明。

特别提醒：exa 已停止维护，以下内容以 eza（exa 的 fork）版本为准。

---

## eza

eza 是一款可用于替换系统默认 ls 的命令。在平时工作中 ls 几乎是用得最多的命令，而 eza 在支持 ls 的基本能力基础上，提供了更丰富的特性。

### 安装

```bash
# macOS
brew install eza

# Debian/Ubuntu（需要手动添加仓库）
sudo apt install eza

# 或者直接从 GitHub 下载二进制
```

### 基础使用

eza 默认行为已经比 ls 友好很多：

```bash
eza                    # 彩色输出，自动区分文件和目录
eza -l                 # 长格式，类似 ls -l
eza -la                # 包含隐藏文件
eza -T                 # 树形显示目录结构
eza -l --git           # 显示 git 状态（哪些文件被修改）
```

### 彩色输出

默认 `eza` 就有颜色区分——目录蓝色、可执行文件绿色、图片文件紫色、符号链接粉色。而 ls 需要 `--color=auto` 才有基础的着色。

### 树形视图

这是 eza 最亮眼的功能，一条命令就能看清项目结构：

```bash
# 查看项目结构
eza -T --git-ignore    # 树形，忽略 gitignore 中的文件

# 限定层级
eza -T -L 2            # 只展开两层
```

### 文件信息

```bash
eza -lbg               # 长格式 + 权限 + 用户 + 分组
eza -lbg --sort=size   # 按文件大小排序
eza -lbg --sort=mod    # 按修改时间排序
eza -lbg --icons       # 显示文件类型图标（配合 Nerd Font）
```

加上 `--icons` 后，每个文件前都有对应的图标——目录 📁、图片 🖼️、代码 🧑‍💻，一目了然。

### 配置别名

建议在 `~/.zshrc` 中添加别名，替换掉默认 ls：

```bash
alias ls="eza"
alias ll="eza -lbg"
alias la="eza -labg"
alias lt="eza -T -L 2"
```

---

## zoxide

zoxide 是基于 frecency（frequency + recency）算法的智能 cd 命令。它会记录你访问过的目录，让你在"猜你想去哪个目录"这件事上做到几乎零思考。用上之后，你会发现自己越来越少打完整的 cd 命令了。

### 安装

```bash
# macOS
brew install zoxide

# 其他系统
curl -sS https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | bash
```

### 配置

将 zoxide 接入 zsh，只需在 `~/.zshrc` 中加一行：

```bash
eval "$(zoxide init zsh)"
```

这会注册 `z` 命令作为 zoxide 的入口。

### 使用方式

安装前先多 cd 到各个目录，让 zoxide 积累数据。之后只要一个关键字就能跳转：

```bash
# 先到各个目录走动，让 zoxide 学习
cd ~/Projects/myapp/src/components
cd ~/Documents/work/reports
cd ~/Downloads

# 之后想去哪里，一个关键字就够了
z comp           # → ~/Projects/myapp/src/components
z reports        # → ~/Documents/work/reports
z down           # → ~/Downloads
```

### 多关键字匹配

```bash
z myapp src      # 多个关键字缩小范围
z work report    # → ~/Documents/work/reports
```

### 核心逻辑

zoxide 的魔力在于它不只是匹配路径名，而是综合考虑了访问频率和最近访问时间：

- 常去的目录 → 权重大
- 最近刚去过 → 加分
- 很久以前的 → 权重衰减

如果你有多个同名目录，常用那个自动排第一。

### 交互模式

如果多个目录匹配，用交互模式选择：

```bash
zi proj    # 弹出 fzf 交互选择器
```

### cd 兼容

zoxide 完全兼容原生 cd——输入完整路径时直接 cd，输入关键字时智能跳转：

```bash
z /etc/nginx    # 完整路径，行为同 cd
z nginx         # 关键字，跳转到匹配的目录
```

---

## bat

bat 是一款支持语法高亮、Git 集成的用于替换 cat 的文件查看命令。它和百度/阿里/腾讯没有关系。

### 安装

```bash
# macOS
brew install bat

# Debian/Ubuntu
sudo apt install bat
```

### 基础使用

```bash
bat main.py                    # 语法高亮 + 行号
bat --line-range 10:20 main.py # 只显示 10-20 行
```

### 语法高亮

默认输出带行号、语法高亮和 Git 修改标记：

```
 1 │ package main
 2 │
 3 │ import "fmt"
 4 │
 5 │ func main() {
 6 │     fmt.Println("Hello")
 7 │ }           ← 行号左侧如果有 + 或 - 表示 git 修改
```

bat 自动检测 100+ 种语言，几乎所有编程语言和配置文件都覆盖了。

### 与管道兼容

当 bat 的输出被 pipe 到其他命令时，自动禁用高亮和分页，行为同 cat：

```bash
bat main.py | head -20    # 自动降级
bat main.py > output.txt  # 也自动降级
bat --plain main.py       # 强制原始输出
```

### 整合到其他工具

```bash
# 结合 ripgrep
rg pattern --context 3 myfile.py | bat

# 结合 diff
diff file1.py file2.py | bat -l diff
```

### 配置

通过 `~/.config/bat/config` 或 `BAT_CONFIG_PATH` 环境变量指定配置文件：

```bash
--theme="Dracula"
--number
--paging=never
```
