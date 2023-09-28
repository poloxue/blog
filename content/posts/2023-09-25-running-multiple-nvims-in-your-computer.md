---
title: "Neovim 配置隔离-实现多语言环境支持"
date: 2023-09-25T15:06:02+08:00
draft: false
comment: true
tags: ["vim", "neovim"]
---

本文将介绍如何实现 Neovim 的配置隔离，实现不同编程语言使用不同的编辑器配置。

## 背景说明

近段时间，一直在学习如何高效使用 Neovim。不断配置的过程中想到，Neovim 是否支持配置隔离，不同用途不同配置。最直接的体现，我希望把 Python 和 Golang 的编辑开发环境的配置隔离。

类似如下效果：

```bash
nvim-golang main.go
nvim-python main.py
nvim-cpp main.cpp
```

> 提到这，不由地想到了 Jetbrain 全家桶，针对不同编程语言开发了各自的 IDE，如 goland，pycharm，webstorm、clion 等。猜测原因，或许是为了多赚钱，另一方面，不同语言一定有个性化配置，隔离能减少耦合。

> 如果是因为想搭建某种语言的编程环境，推荐阅读：[基于 LunarVim 搭建不同编程语言 IDE](https://www.poloxue.com/posts/2023-09-27-start-an-ide-using-lunarvim/)

如何实现呢？进入正题吧。

> 几年前，写过一篇关于 "Golang 多环境管理 GVM" 的文章。本质上，要实现这种多环境隔离，一般都是通过环境变量实现。查了些资料，Neovim 其实也不例外。

## 方案 1：基于 XDG 配置

Neovim 的目录遵循 XDG 目录规范。具体是什么意思呢？ 

XDG 本质是一套规范，定义了一组环境变量，用于说明应用程序储存信息目录的一套标准。熟悉 Linux 的朋友应该了解，我们以往一直习惯于把应用的配置以 .xxx 的形式放在用户的 `$HOME` 目录，导致 `$HOME` 下的点隐藏文件泛滥，而这套规范的出现，使我们轻易实现目标。

就以 Neovim 为例：

Neovim 的配置文件存放默认存放在 `$XDG_CONFIG_HOME/nvim`，数据目录默认在 `$XDG_DATA_HOME/nvim`，状态数据目录默认在 `$XDG_STATE_HOME/nvim`，缓存数据目录默认在 `$XDG_CACHE_HOME/nvim`。

通过修改 XDG 环境变量，即可实现环境隔离。

如下 nvim-golang 的启动脚本：

```bash
#!/bin/bash

export XDG_CONFIG_HOME=$HOME/.config/nvim-golang
export XDG_DATA_HOME=$HOME/.local/share/nvim-golang
export XDG_STATE_HOME=$HOME/.local/state/nvim-golang
export XDG_CACHE_HOME=$HOME/.cache/nvim-golang

nvim $@
```

注：XDG 变量的修改范围只能在命令脚本内，不可影响 XDG 全局配置。

现在的 nvim 目录情况：

```bash
$HOME/.config/nvim --> $HOME/.config/nvim-golang/nvim
$HOME/.local/share/nvim --> $HOME/.local/share/nvim-golang/nvim
$HOME/.local/state/nvim --> $HOME/.local/state/nvim-golang/nvim
$HOME/.cache/nvim --> $HOME/.cache/nvim-golang/nvim
```

假设，配置一个 nvim-golang 的编辑器，以 LunarVim 的作者提供的 nvim-basic-ide 配置为例。

下载配置到指定目录：

```bash
git clone https://github.com/LunarVim/nvim-basic-ide.git $HOME/.config/nvim-golang/nvim
```

假设，上述启动脚本文件名为 `nvim-golang`，执行脚本 `nvim-golang main.go` 编辑文件，初次进入会下载安装 `nvim-basic-ide` 中的插件。

## 方案二：NVIM_APPNAME

或许是官方也知道了配置隔离的价值。从 Neovim 0.9.0 开始，Neovim 默认支持一个环境变量 `NVIM_APPNAME` 隔离环境配置。

以 nvim-python 为例，我们只需通过 `NVIM_APPNAME=nvim-python nvim main.py`，即可改变 Nvim 的目录。

此时目录：

```bash
$HOME/.config/nvim --> $HOME/.config/nvim-golang
$HOME/.local/share/nvim --> $HOME/.local/share/nvim-golang
$HOME/.local/state/nvim --> $HOME/.local/state/nvim-golang
$HOME/.cache/nvim --> $HOME/.cache/nvim-golang
```

相对于方案一而言，无需在 nvim-python 追加子目录 nvim ，启动脚本 `nvim-python` 的内容只要一行代码：

```bash
NVIM_APPNAME=nvim-python nvim @
```

或者更简洁的实现，通过 alias 别名：

```bash
alias nvim-python="NVIM_APPNAME=nvim-python nvim"
```

将 `nvim-basic-ide` 下载到 `~/.config/nvim-python` 下：

```bash
git clone https://github.com/LunarVim/nvim-basic-ide.git $HOME/.config/nvim-python
```

与方案一的效果一致，后面针对 python 环境的定制就可以都在这个环境下进行了。

## 方案三：混合使用

方案一和二都有个小缺点，首先一方案 nvim-golang 总要追加一个子目录，而方案二呢？在 ~/.config 下会创建多个 nvim 相关配置。

那么，有没有一种方法，能将 nvim 的不同配置集合到一个目录之下，统一管理，效果以类似于:


```bash
$HOME/
  - .config/
    ...
    - nvims
      - golang
      - python
    ...
```

简单！两者结合即可：

首先，将 XDG 的配置统一下移到 `nvms` 下，什么意思呢？如原来的 `XDG_CONFIG_HOME` 是 `$HOME/.config/`，现在是 `$HOME/.config/nvims`，其他变量类似：

脚本如下：

```bash
export XDG_CONFIG_HOME=$HOME/.config/nvims
export XDG_DATA_HOME=$HOME/.local/share/nvims
export XDG_STATE_HOME=$HOME/.local/state/nvims
export XDG_CACHE_HOME=$HOME/.cache/nvims
```

接着，通过 `NVIM_APPNAME` 作为 `nvims` 的子目录。如 `nvim-golang` 的命令实现：

```bash
NVIM_APPNAME=golang nvim $@
```

现在只要将 `nvim-basic-ide` 的配置下载在 `$HOME/.config/nvims/golang` 下。

```bash
git clone https://github.com/LunarVim/nvim-basic-ide.git $HOME/.config/nvims/golang
```

搞定！

完整 `nvim-golang` 脚本如下：

```bash
export XDG_CONFIG_HOME=$HOME/.config/nvims
export XDG_DATA_HOME=$HOME/.local/share/nvims
export XDG_STATE_HOME=$HOME/.local/state/nvims
export XDG_CACHE_HOME=$HOME/.cache/nvims

NVIM_APPNAME=golang nvim $@
```

启动 `nvim-golang` 查看效果。

## 插件复用

最后，还有一个问题，即配置的过程中，可能已经有朋友发现，插件会重复多次下载。这是因为，默认插件在下载位置位于 `$XDG_DATA_HOME/$NVIM_APPNAME` 下，故而不同的 `$XDG_DATA_HOME` 或 `NVIM_APPNAME` 都会导致下载插件所在位置的不同。

但插件管理器一般是支持配置插件下载路径的，以 lazy 为例，`nvim-basic-ide` 中 `init.lua` 中的代码如下：

```bash
local lazypath = vim.fn.stdpath "data" .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system {
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  }
end
vim.opt.rtp:prepend(lazypath)

-- ...

require("lazy").setup("user", {
  root = vim.fn.stdpath("data") .. "/../lazy",
  ...
})
```

通过修改 `lazypath` 可以和 `setup` 时设置 `root` 变量实现多环境共享插件。

那么，插件的问题解决了。

尝试实操下。

首先，下载 `nvim-basic-ide` 到 nvims 的 `~/.config/nvims/golang/` 目录。

```bash
git clone https://github.com/LunarVim/nvim-basic-ide.git $HOME/.config/nvims/golang
```

将 `$HOME/.config/nvims/golang/lua/Lazy.lua` 中的 `lazypath` 更新为：

```lua
local lazypath = vim.fn.stdpath "data" .. "/../lazy/lazy.nvim"
```

同样方法配置 nvim-python，将 `nvim-golang` 配置拷贝一份，即 `~/.config/nvims/golang` 拷贝到 `~/.config/nvims/python`。同时，将 `nvim-golang` 启动脚本拷贝一份为 `nvim-python`，`NVIM_APPNAME` 修改为 `python`。

如下所示：

```bash
#!/bin/bash

export XDG_CONFIG_HOME=$HOME/.config/nvims
export XDG_DATA_HOME=$HOME/.local/share/nvims
export XDG_STATE_HOME=$HOME/.local/state/nvims
export XDG_CACHE_HOME=$HOME/.cache/nvims

NVIM_APPNAME=python nvim $@
```

测试效果，启动 nvim-python 已无需重新加载插件。

## 代码复用

另外一个优化点，是否可以将 `nvim-basic-ide` 的能力作为基础共享，在此基础上配置 `goalng` 和 `python` 各自的配置。

额？好像 `LunarVim` 支持这个能力，它已经把用户配置从核心 IDE 基础配置中摘除出来了。

它的基本思路是在核心配置中引入用户定义配置。

简单来说就是主配置目录下的 `init.lua` 通过一个变量 `LUNARVIM_CONFIG_DIR` 实现 `require` 用户配置 `config.lua` 实现多环境。实现代码在 [bootstrap.lua](https://github.com/LunarVim/LunarVim/blob/master/lua/lvim/bootstrap.lua) 文件中。

以 golang 环境为例，一个别名搞定：

```bash
alias lvim-golang="LUNARVIM_CONFIG_DIR=${HOME}.config/lvims/golang lvim @"
```

我们在 `~/.config/lvims/golang/config.lua` 实现针对 Golang 的自定义配置。

有兴趣的话，可以实现下，这套思路也不错。

> 特别补充：这个思路可以完全舍弃前面提到的几种配置隔离方案，不需要修改 XDG 变量或者 NVIM_APPNAME。它通过唯一的配置入口 require 不同环境（LUNARVIM_CONFIG_DIR）下的 `config.lua`，从而实现复用核心配置，同时可以自定义个性化需求。

> Lunarvim 针对不同编程语言的配置方法，可查看 [基于 LunarVim 搭建不同编程语言 IDE](https://www.poloxue.com/posts/2023-09-27-start-an-ide-using-lunarvim/)

## 最后

关于配置隔离的思路，到此就已经全部介绍完了。

这个思路不仅仅可用于多语言环境的隔离，对于市面上有很多别人维护的配置，利用这个思路都能快速下载体验。

如下是市面上流行的一些集成配置：

- [kickstart](https://github.com/nvim-lua/kickstart.nvim)，所有配置一个文件搞定；
- [neovim-basic-ide](https://github.com/LunarVim/nvim-basic-ide): 本文测试所用，LunarVim 开发者维护的一个基础配置；
- [LazyVim](https://github.com/LazyVim/LazyVim.git): 提供 IDE 所需的基本配置；
- 还有 [NvChad](https://github.com/NvChad/NvChad) 、[AstroNvim](https://github.com/AstroNvim/AstroNvim) 等，甚至一些个人也会开源自己的配置；

有兴趣都可以尝试下有什么差异。


博文地址：[Neovim 配置隔离-实现多语言环境支持](https://www.poloxue.com/posts/2023-09-25-running-multiple-nvims-in-your-computer/)
