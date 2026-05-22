---
date: 2024-02-25
title: "基础插件"
weight: 2
---

# 基础插件

重点来了，接下来我们一起来看看 zsh 的效率神器 - 插件能力吧。本文先给大家推荐 5 款常用的插件。oh-my-zsh 提供的所有内置插件，都可以在仓库 [ohmyzsh/ohmyzsh/plugins](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins) 中找到，每个插件都有相应的介绍文档。

本文介绍的 5 个插件：

| 插件 | 作用 |
|------|------|
| `git` | 大量 git 别名，敲 3 个字母代替 10 个 |
| `sudo` | 双击 Esc 在命令前加 sudo |
| `extract` | `x` 解压任何格式的压缩包 |
| `web-search` | 终端里直接搜索搜索引擎 |
| `colored-man-pages` | man 页面语法高亮 |

启用插件很简单，编辑 `~/.zshrc`，找到 `plugins=(git)` 这一行，加入需要的插件名：

```bash
plugins=(git sudo extract web-search colored-man-pages)
```

然后执行 `source ~/.zshrc` 生效。

## git

推荐优先级最高，**必装插件**。它提供了一百多个 git 别名，日常操作从 3-5 个字母变成 1-2 个：

| 别名 | 原命令 | 说明 |
|------|--------|------|
| `gst` | `git status` | 查看状态 |
| `ga` | `git add` | 添加文件 |
| `gcmsg` | `git commit -m` | 提交 |
| `gp` | `git push` | 推送 |
| `gl` | `git pull` | 拉取 |
| `gco` | `git checkout` | 切换分支 |
| `gb` | `git branch` | 分支管理 |
| `gd` | `git diff` | 查看差异 |
| `glog` | `git log --oneline --decorate --graph` | 精美日志 |
| `gsta` | `git stash` | 暂存 |
| `gcf` | `git config --list` | 查看配置 |

有了这些别名，从此告别敲完整的 git 命令。

## sudo

有的命令敲完了才发现忘了加 `sudo`——这个插件就是为了解决这个问题。在任意命令上双击 `Esc`，自动在命令前加上 `sudo`：

```bash
# 输入
systemctl restart nginx
# 双击 Esc 后变成
sudo systemctl restart nginx
```

不用回退到行首加 sudo，省一次 Home 键。

## extract

解压文件最烦的就是记参数——`tar -xzf` 还是 `tar -xf`？`unzip` 还是 `unrar`？这个插件提供一个统一的 `x` 命令：

```bash
x file.tar.gz      # 自动识别格式解压
x file.zip         # 同上
x file.rar         # 同上
x file.tar.xz      # 同上
```

它支持 tar.gz、tar.bz2、tar.xz、zip、rar、7z 等几乎所有常见压缩格式，无需记忆参数。

## web-search

在终端里直接搜索，不用切到浏览器：

```bash
# 搜索 Google
google how to exit vim

# 搜索 GitHub
github zsh plugin

# 搜索 Wikipedia
wiki zshell

# 指定搜索引擎
open "https://stackoverflow.com/search?q=zsh+plugin"
```

## colored-man-pages

给 `man` 命令加上语法高亮，让手册页面更容易阅读。开启后，代码块、选项、关键字都会有不同的颜色标识，对比黑白灰的原始 man 页面，体验提升一个档次。

## 小结

以上 5 个插件是 oh-my-zsh 的基础入门推荐。`git` 和 `sudo` 是日常必装，`extract` 和 `web-search` 能省去很多重复记忆，`colored-man-pages` 让读文档更友好。下一篇继续介绍更多基础插件。
