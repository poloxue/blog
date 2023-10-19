---
title: "zsh 安装与主题，推荐 7 个提升效率的 zsh 插件"
date: "2023-10-16T15:00:06+08:00"
draft: true
comment: true
tags: ["zsh"]
---

本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件。

## 安装

使用 macOS 的用户，zsh 从 2019 已经是默认 shell，无需安装。如果是更早版本，或其他 Linux 发行版本，则这个步骤无法跳过。

不同系统的安装命令如下： 

MacOS 系统

```bash
brew install zsh；
```

Debian

```bash
apt install zsh
```

Redhat

```bash
```

- 说明：2019 开始，macOS 默认 shell 从 bash 切换到 zsh，该步骤可省略。 推荐阅读：[What is Zsh? Should You Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in.)介绍了为什么推荐使用 zsh，以及为什么从 2019 macOS 将默认的 shell 从 bash 切换到 zsh。


oh-my-zsh
- 说明：oh-my-zsh 是zsh 用于管理配置的轻量级框架。
- 安装：sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
- 生效：
  - 重启 iTerm2；
  - 或执行 `source ~/.zshrc`
  - 即可生效配置。
- 主题
  - oh-my-zsh 自带主题，查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)。
  - 体验
    - 编写 `.zshrc`，更新 `ZSH_THEME="agnoster"`；
    - 执行 `source ~/.zshrc` 查看效果；

配置插件
- 内置插件
  - 文档仓库：https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins
  - git，内置插件
  - web-search，搜索引擎搜索；
  - jsontools，用于处理 json 数据；
  - z，目录快速跳转；
  - vi-mode，使用 vi 模式编辑命令行；
- 三方插件：
  - 安装：
    ```zsh
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
    ```
  - zsh-syntax-highlighting
  - zsh-autosuggestions
- 配置：~/.zshrc
  - 配置插件： 
    ```zsh
    plugins=(git web-search jsontools z vi-mode zsh-syntax-highlighting zsh-autosuggestion)
    ```
  - 自动补全：
  ```zsh
   bindkey '^_' autosuggest-accept```
  - 说明：
    - <Ctrl+/> 接受 auto-suggestion 的补全建议

