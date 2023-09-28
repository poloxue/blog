---
title: "基于 LunarVim 搭建不同编程语言 IDE"
date: "2023-09-27T15:22:23+08:00"
draft: false
comment: true
tags: ["neovim", "vim"]
---

本文介绍，如何基于 LunarVim 搭建不同编程语言的 Neovim IDE 开发环境。

## 前言

本文将用几行命令快速安装 Neovim IDE，完成不同编程语言的环境搭建。尽量不涉及到自定义配置，将完全基于 LunarVim 作者维护的配置实现。

两个 Github 核心仓库，分别是：

- [lunarvim/lunarvim](https://github.com/lunarvim/lunarvim)，是 LunarVim 的核心仓库，集成配置 IDE 所需的核心能力；
- [lunarvim/starter.lvim](https://github.com/lunarvim/starter.lvim)，这个仓库是 Lunarvim 针对不同编程语言的配置实现；

starter.lvim 以分支形式保不同语言的配置，具体自行查看仓库。

为了测试方便，介绍 LunarVim 提供的一个能力，通过 Lunarvim 通过 `LUNARVIM_CONFIG_DIR` 变量决定配置文件目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-03.png)

接下来的测试，我会将不同语言的配置，放到不同的目录中。

> 如果希望一个配置支持大部分语言，则要将这些配置合并，进行配置自定义，对 Lunarvim 的 `configlua` 代码进行大的改动，不易维护。

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

## 初步体验

体验下 Lunarvim  默认配置，与语言无关的能力。

### 目录浏览

使用 "<空格>+e" 显示文件树，再次 "<空格>+e" 关闭，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-07.gif)

### 查找文件

使用 “<空格>+f”，查看文件，如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-06.gif)

### 查找符号

输入 "<空格>+lS"，全部符号搜索，如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-08.gif)

### 重构变量名

输入 "<空格>+lr"，将 'anyMethods' 更新为 'anyMethodList'，如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-09.gif)

### 类型引用查找

输入 "gr"，查找 "RouterGroup" 被引用的地方。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-10.gif)

### 定义跳转

输入 "gd"，跳转到 "RouterGroup" 定义。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-11.gif)

### Git 支持

内置 lazygit 处理复杂事务，同时内置对部分 ShortCut 和简单的 Git 命令的进行了绑定。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-27-start-an-ide-using-lunarvim-12.gif)


LunarVim 默认支持的能力还是挺丰富的。

详细内容可查看它的文档，或者 "<空格>" 显示 whch-key 菜单，自行探索。深入了解的话，最好是阅读 Lunarvim 的 lua 配置源码。如果能独立配置 Neovim，再看它的源码会有更好的体悟。

## 配置

前面已经介绍 LunarVim 和 starter.lvim 的关系。实操的话，如何配置呢？

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

错误提示，自动补全能力，都已经支持的很棒了。

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

其他语言 IDE 的安装就不详细说了，类似步骤。另外，starter.lvim 支持编程语言很多，按需要取用。

 ## 最后说明

本文只是快速使用体验教程，如希望更深入了解，可自行查看配置。诸如了解它的快捷键如何使用，只有直接看配置才能知道。

另外，这里的配置除了支持 lsp，language server，基本也都支持 dap，即调试能力。

由于，这些配置的使用细则都没有提供文档，而且既然已经用 neovim 了。至少还是要能看懂和写写配置的，不然是没有办法用好 Neovim 的，更谈不上，个人偏好的配置自定义。

