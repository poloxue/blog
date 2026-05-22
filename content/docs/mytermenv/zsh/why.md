---
date: 2024-02-11
title: "为什么选择 zsh ？"
weight: 2
---


开始前，先问为什么，知其然，要知其所以然，是个好习惯。所以，为什么要用 zsh 呢？

大家最熟悉的 shell 解释器，肯定是 bash。zsh（Z Shell）相对于 bash（Bourne Again Shell）有哪些优势呢？

## 开箱即用的体验提升

zsh 的默认行为就已经比 bash 友好很多：

### 智能补全

在 bash 里按 Tab 只是简单补全文件名。但在 zsh 中：

- 输入 `cd` 然后按 Tab→ 列出目录，再次 Tab 可进入子目录选择
- 输入 `git ` 按 Tab→ 列出所有 git 子命令
- 输入 `ssh ` 按 Tab→ 列出 `~/.ssh/config` 中的主机
- 输入 `kill ` 按 Tab→ 列出进程列表

### 拼写纠正

你输入 `cd Documetns`，zsh 会问你是不是想进 `Documents`？输入 `sl` 会提示是否要执行 `ls`？这个功能拯救过我不计其数的笔误。

### 更强大的通配符

```bash
# 递归匹配所有 .py 文件（** 表示任意层目录）
ls **/*.py

# 排除某些文件
ls *.txt~readme.txt

# 匹配特定范围
ls file[1-10].txt
```

### 目录别名

不用 `cd` 就能切换目录：

```bash
# 直接输入目录名即可进入
/usr/local/bin

# 使用 ~ 加关键字快速跳转
~doc    # 等价于 cd ~/Documents
```

## 极致可定制

zsh 的提示符（prompt）支持完善的样式定制，甚至可以右侧显示信息：

```bash
# 在 .zshrc 中设置右侧提示符显示 git 分支
RPROMPT='$(git_branch)'
```

结合 Powerlevel10k 主题，你能得到一个信息丰富又美观的提示符——显示命令执行时间、git 状态、Python 虚拟环境、后台 Job 数量等。

## 丰富的插件生态

oh-my-zsh 社区贡献了 300+ 内置插件，几乎覆盖了所有常用工具：

- `git` — git 命令别名，`gst`= `git status`, `gco`= `git checkout`
- `docker` — docker 命令补全
- `pip` — pip 补全
- `brew` — Homebrew 补全
- `sudo` — 双击 Esc 在命令前加 sudo

不用自己去 `.bashrc` 里写一堆 `alias`，装个插件就全有了。

## 兼容性

zsh 在语法上几乎完全兼容 bash——你的 `.bashrc`、`.bash_profile` 里的内容可以直接搬到 `.zshrc` 里用。迁移成本极低，但收益巨大。

## 小结

如果你还在用 bash，切换到 zsh 是性价比最高的终端效率提升——零成本、零风险，但每天都能感受到的体验提升。下一章我们通过 oh-my-zsh 把 zsh 武装起来。
