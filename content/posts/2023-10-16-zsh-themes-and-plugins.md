---
title: "我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件"
date: "2023-10-16T15:00:06+08:00"
draft: true
comment: true
tags: ["zsh"]
---

{{< video bb_id=619786097 >}}

本教程将主要介绍 zsh 的安装、主题，以及介绍 7 提升效率的 zsh 插件。

## 安装

对于 macOS 用户，zsh 从 2019 已经是默认 shell，无需安装。该步骤可省略。推荐阅读：[What is Zsh? Should You Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in.)介绍了为什么推荐使用 zsh，以及为什么从 2019 macOS 将默认的 shell 从 bash 切换到 zsh。

当然如果是更早版本，或其他 Linux 发行版本，则这个步骤无法跳过。

不同系统的安装命令如下： 

MacOS 系统

```bash
brew install zsh
```

Debian

```bash
apt install zsh
```

Centos

```bash
yum install -y zsh
```

Arch Linux

```bash
pacman -S zsh
```

Fedora

```bash
dnf install zsh
```

安装完成，要将 zsh 设置为默认 shell，命令如下所示：

```bash
chsh -s /bin/zsh
```

通过如下命令：

```bash
echo $SHELL
```

检查是否设置成功。

## oh-my-zsh

oh-my-zsh 是zsh 用于管理配置的轻量级框架。下面将用它快速配置 zsh。

安装命令如下：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 主题

oh-my-zsh 提供了许多内置主题，查看 [themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)。

打开 ~/.zshrc 配置文件，更新主题配置：

```bash
ZSH_THEME="agnoster"`
```

执行 `source ~/.zshrc` 生效，查看效果。

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

