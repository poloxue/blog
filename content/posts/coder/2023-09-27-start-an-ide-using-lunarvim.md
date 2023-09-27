---
title: "基于 LunarVim 搭建不同编程语言 IDE"
date: "2023-09-27T15:22:23+08:00"
draft: false
comment: true
tags: ["neovim", "vim"]
---

本文介绍，如何基于 LunarVim 实现不同编程语言的 Neovim IDE。

动图效果，如下所示：

Golang

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-04.gif)

Python

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-05.gif)

## 前言

本文将用几行命令快速安装 Neovim IDE，完成不同编程语言的环境搭建。尽量不涉及到自定义配置，将完全基于 LunarVim 作者维护的配置实现。

两个 Github 核心仓库，分别是：

- [lunarvim/lunarvim](https://github.com/lunarvim/lunarvim)，是 LunarVim 的核心仓库，集成配置 IDE 所需的核心能力；
- [lunarvim/starter.lvim](https://github.com/lunarvim/starter.lvim)，这个仓库是 Lunarvim 针对不同编程语言的配置实现；

starter.lvim 以分支形式保不同语言的配置，具体自行查看仓库。

为了测试方便，介绍 LunarVim 提供的一个能力，通过 Lunarvim 通过 `LUNARVIM_CONFIG_DIR` 变量决定配置文件目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-03.png)

> 注：图中变量名写错了，懒的改了。

接下来的测试，我会将不同语言的配置，放到不同的目录中。

> 要将这些配置合并，还需要自定义配置，大改 Lunarvim 的 `configlua` 代码，且不易维护。

## 安装

LunarVim 提供了安装脚本，使用如下命令安装即可。

```bash
LV_BRANCH='release-1.3/neovim-0.9' bash <(curl -s https://raw.githubusercontent.com/LunarVim/LunarVim/release-1.3/neovim-0.9/utils/installer/install.sh)
```

安装过程中要下载一些依赖，如 `pynvim`，`cargo` 之类的，如果已经安装可选择 `no`。

> 注：部分语言环境和命令要提前安装。如 `python`，`make`, `git` 等

如果要 dev icon 支持，安装 Nerd 字体，macOS 安装命令如下：

```bash
brew tab homebrew/cask-fonts
brew install --cask font-hack-nerd-font
```

安装成功，将终端字体更新为 `Hack Nerd Font` 相关字体。

如下所示，以 iTerm2 为例：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-02.png)

现在执行 `lvim`，将看到 `lazy.nvim` 开始自动下载插件，如下是效果图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-01.png)

## 配置

前面已经介绍 LunarVim 和 starter.lvim 的关系。

### Golang

首先，以 go-ide 为例，使用如下命令启动 lvim：

```bash
alias lv-go="LUNARVIM_CONFIG_DIR=~/.config/lvims/golang lvim "
```

将 starter.lvim 的 `go-ide` 分支下载到 `~/.config/lvims/golang/` 目录下。

```bash
git clone https://github.com/Lunarvim/starter.lvim --branch go-ide ~/.config/lvims/golang
```

下载完成，记得在原有的 config.lua 中的新增一行配置，如下所示;

```lua
lvim.format_on_save.enabled = true
```

实现保存后的通过 `goimports` 和 `gofumpt` 实现代码的自动格式化。

运行 `lv-go` 开启 Go 项目。

首次进入，通过 `Mason` 安装一些 Go 相关的工具，如下：

```bash
:MasonInstall gopls golangci-lint-langserver delve goimports gofumpt gomodifytags gotests impl
```

体验下效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-04.gif)

## Python


配置 python-ide，使用如下命令启动 lvim：

```bash
alias lv-py="LUNARVIM_CONFIG_DIR=~/.config/lvims/python lvim "
```

将 starter.lvim 的 `go-ide` 分支下载到 `~/.config/lvims/python/` 目录下。

```bash
git clone https://github.com/Lunarvim/starter.lvim --branch python-ide ~/.config/lvims/python
```

运行 `lv-py` 开启 Python 项目。效果图如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-05.gif)

php IDE 的安装就不详细说了，类似步骤。另外，starter.lvim 支持编程语言很多，按需要取用。

 ## 最后说明

本文只是快速使用体验教程，如希望更深入了解，可自行查看配置。诸如了解它的快捷键如何使用，只有直接看配置才能知道。另外，这里的配置基本都支持 dap 能力，即调试能力。

由于，这些配置的使用细则都没有提供文档，而且既然已经用 neovim 了。至少还是要能看懂和写写配置的，不然是没有办法用好 Neovim 的，更谈不上，个人偏好的配置自定义。

