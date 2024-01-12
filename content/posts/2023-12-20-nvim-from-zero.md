---
title: "Neovim 从 0 快速配置: QuickStart Guide"
date: 2024-12-20T15:06:02+08:00
draft: true
comment: true
tags: ["vim", "neovim"]
---

本文将介绍如何从 0 配置 Neovim。

为了快速配置，本文中的全部配置均在 `init.lua`，无目录布局。基于它，后续文章再逐步展开，介绍具体细节。

## 前言

Neovim 是 Vim 的衍生版本，相较于传统 vim，它支持通过 `lua` 脚本配置其行为。

## 快速安装

建议版本至少不低于 vim 0.9.0，我的版本是 Macos 0.9.2。

安装命令，如 MacOS：

```bash
brew install neovim
```

注：不同的 Linux 发行版一般会默认提供 Neovim 安装包。如果默认版本过低，建议从 [GitHub Release 页面](https://github.com/neovim/neovim/tags) 下载安装。

下载安装，如下所示：

```bash
wget https://github.com/neovim/neovim/releases/download/v0.9.2/nvim-linux64.tar.gz
tar zxvf nvim-linux64.tar.gz
mv ./nvim-linux64/bin/nvim /usr/local/bin
```

运行 `nvim` 检查是否安装成功。

## 配置文件

Neovim 支持两种方式配置，基于 vimscript 和基于 lua 脚本。

- vimscript 配置文件 `init.vim` 位于 `$HOME/.config/nvim/init.vim`；
- lua script 配置文件 `init.lua` 位于 `$HOME/.config/nvim/init.lua`;

要注意的是，两种方式无法同时生效。

> 注明：Neovim 的目录规范遵循 XDG 规范，对于 Neovim 而言，如下：
> - config 文件位于 `$HOME/.config/nvim`；
> - data 目录位于 `$HOME/.local/share/nvim`（如插件）；
> - cache 目录位于 `$HOME/.cache/nvim`；
>
> 了解更多，建议阅读规范：[XDG Base Directory specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)。

如果希望继续使用以往的 vim 配置，即 `.vimrc`，可在 `init.vim` 中引入。`init.vim` 实现如下：

```vim
set runtimepath^=~/.vim runtimepath+=~/.vim/after
let &packpath=&runtimepath
source ~/.vimrc
```

或通过 lua 脚本将迁移原配置，如 option，通过 Neovim 的 Lua API 接口 `vm.opt.{option}`（或 vim.o 迁移。

```vim
set number 等同 vim.opt.number = true
``````

> 注：Neovim 中 Lua 的使用，直接输入 `:h lua-guide` 查看文档。如果对 lua 的基础使用不熟悉，推荐[菜鸟教程](https://www.runoob.com/lua/lua-tutorial.html)或 [Learn X in Y minutes: Where X = Lua](https://learnxinyminutes.com/docs/lua/)。

正文开始，所有代码都在 `init.lua` 中。

## 基础配置

打开 `~/.config/nvim/init.lua`， 添加基础配置，包括选项和按键，即 options 和 keymap。

### 配置选项

Neovim 通过 `vim.o` 配置一些基础选项，实现 `vimrc` 中的 `set xxx`，如下所示：

```lua
-- 显示行号
vim.o.number = true

-- 默认缩进配置宽度
vim.o.shiftwidth = 2
vim.o.expandtab = true
vim.o.tabstop = 2
vim.o.softtabstop = 2

-- 搜索相关
vim.o.hlsearch = true
vim.o.ignorecase = true
vim.o.smartcase = true

vim.o.completeopt = 'menuone, noselect'
```

完整代码，请查看：

### 按键绑定

修改 leader key 为 `<空格>`，默认 leader key 是 `/`，操作难度高。

```lua
vim.g.mapleder = ' '
vim.g.localleader = ' '
```

将 mapleader 和 localmapleader 设置为 `<空格>`，其中，`vim.g.{variable}` 可用于设置 vimscript 中的全局变量。

禁用 `<空格>` 的默认行为

Vim 中，如果空格没有成功映射到快捷键，默认行为是 forward，即向前进一步，如下配置可禁用这个行为。

```lua
local keymap = vim.api.nvim_set_keymap
keymap("", "<Space>", "<Nop>", { noremap = true, silent = true })
```

## 插件管理

