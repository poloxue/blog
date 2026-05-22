---
date: 2024-03-10
title: "高级插件"
weight: 4
---

# 高级插件

我们再来了解 4 个非 oh-my-zsh 内置插件，它们分别是 zsh-syntax-highlighting、zsh-autosuggestions、zsh-history-substring-search 和 you-should-use。这些插件由 zsh 社区开发。

开始介绍前，先将它们全部安装配置完成。

## 安装

这 4 个插件需要手动从 GitHub 克隆到 `~/.oh-my-zsh/custom/plugins/` 目录：

```bash
# 语法高亮
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 自动建议
git clone https://github.com/zsh-users/zsh-autosuggestions.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# 历史命令搜索
git clone https://github.com/zsh-users/zsh-history-substring-search.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search

# 别名提示
git clone https://github.com/MichaelAquilina/zsh-you-should-use.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/you-should-use
```

然后在 `~/.zshrc` 的 plugins 中加入：

```bash
plugins=(
  ...  # 之前的基础插件
  zsh-syntax-highlighting
  zsh-autosuggestions
  zsh-history-substring-search
  you-should-use
)
```

执行 `source ~/.zshrc` 生效。

## zsh-syntax-highlighting

**必装！** 这个插件会在你输入命令的同时进行语法高亮：

- 合法命令 → 绿色
- 非法命令 → 红色
- 存在的路径 → 带下划线
- 选项 → 紫色

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-10-zsh-syntax-highlighting.gif)

输入 `docker ps`，`docker` 变绿；输入 `docker p` 还没敲完时灰色，敲完后变绿——即时反馈，减少低级错误。

## zsh-autosuggestions

**同样必装！** 它会根据你的历史记录，在输入时以灰色提示你可能想打的完整命令，按 `→` 或 `Ctrl + F` 直接补全：

```bash
$ docker compose up -d    # 之前执行过的命令
$ doc                     # 输入时，灰色显示 →ker compose up -d
                          # 按 → 自动补全
```

用久了你会发现，日常 80% 的命令都是重复的—— `git push`、`docker compose up`、`ssh server`——这个插件让你几乎不用完整打字。

## zsh-history-substring-search

oh-my-zsh 自带的 `history` 插件需要输入关键字再按方向键向上搜索。而这个插件更自然：

| 快捷键 | 动作 |
|--------|------|
| `Ctrl + P` / `↑` | 向上匹配历史 |
| `Ctrl + N` / `↓` | 向下匹配历史 |

输入 `git p` 然后按 `↑`，它会依次匹配 `git push`、`git pull`、`git log -p`……而且高亮匹配的关键字部分，方便定位。

## you-should-use

这个插件有点意思——它会在你输入一个命令的完整形式时，提示你还有别名可用：

```bash
$ git status
# You should use: gst

$ docker ps
# You should use: dps   （如果有配置 docker 插件别名）
```

它是学习 aliases 的绝佳工具——安装后正常用原生命令，插件会提醒你有哪些更短的别名可用，久而久之就自然记住了。

## 小结

这 4 个插件配合使用，体验提升非常明显：

- **语法高亮** → 输错命令立刻知道
- **自动建议** → 重复命令不用再打
- **历史搜索** → 往事如风，上下翻一翻就回来了
- **别名提示** → 被动学习最有效率

安装完它们，终端的智能程度已经远超默认 bash 了。
