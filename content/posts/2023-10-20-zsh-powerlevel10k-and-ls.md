---
title: '我的终端环境：zsh 主题 powerlevel 安装与 ls 命令的替代方案'
date: "2023-10-20T10:25:36+08:00"
draft: true
comment: true
courses: ["我的终端环境"]
---

本教程介绍如何安装 zsh 主题 powerlevel10k。

## 什么是 powerlevel10k?

Powerlevel10 是一款 zsh 的主题，它强调性能、灵活性和开箱即用。我们之前介绍的一些主题，如果使用 powerlevel10k 同样能配置出类似的效果。

## 依赖安装

安装 powerlevel10k 前，先安装它依赖的字体：[NerdFont 文档](https://github.com/ryanoasis/nerd-fonts#font-installation)。不同系统的安装方法，可查看它的文档。

MacOS 的话，可直接通过 Homebrew 快速安装：

```bash
brew tap homebrew/cask-fonts
brew install font-hack-nerd-font
```

安装完成，配置终端字体，进入 iTerm2 Settings -> Profiles -> Text -> Font -> MesloLGS NF

## 安装 powerlevel10k

通过如下命令 下载：

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

配置 `~/.zshrc`，启用 `powerlevel10k` 主题：

```bash
ZSH_THEME="powerlevel10k"
```

执行 `source ~/.zshrc` 生效配置。

## 配置 powerlevel10k

执行 `source ~/.zshrc` 周，终端进入到 powerlevel10k 的配置向导：

如下所示，配置开始时，我们要先回答几个问题才能进入到真正的自定义阶段:

诸如，这是钻石吗？这是锁吗？这是 Debian 图标？等等，我都选择了 `Yes`。

  - 提示符风格：
    - Lean
    - Classic
    - Rainbow
    - Pure
  - 字符编码
    - Unicode
    - ASCII
  - 时间显示
    - No
    - 24 小时制
    - 12 小时制
  - 提示分隔符
    - 箭头 > 
    - 垂直 |
    - 倾斜 /
    - 圆弧 )
  - 提示头部
    - 箭头
    - 模糊
    - 倾斜
    - 圆弧
  - 提示尾部
    - 平整
    - 模糊
    - 箭头
    - 倾斜
    - 圆弧
  - 提示符高度
    - 一行
    - 两行
  - 提示符间隙
    - 紧凑
    - 稀疏
  - 图标
    - 少图标
    - 多图标
  - 提示符流
    - 简洁
    - 流畅
  - 启用瞬间提示
    - 是
    - 否
  - 即时提示模式
    - 关闭
    - 静默启用
    - 启用
- 手动配置
  - .p10k.zsh
  - 配置左右提示信息

文件列表
- 说明：ls 的替代方案 exa 或 colorls。
- [exa](https://github.com/ogham/exa):
  - 安装：brew install exa
- colorls 安装：sudo gem install colorls
- 替代 ls
  ```zsh
  if [ -x "$(command -v exa)" ]; then
    alias ls=exa --icons
  fi
  ````
- colorls
  ```zsh
  if [ -x "$(command -v colorls)" ]; then
    alias ls="colorls"
    alias la="colorls -al"
  fi
  ```
