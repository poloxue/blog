---
title: "从 0 将 Neovim 打造成一个 IDE"
draft: true
comment: true
tags: ["vim", "neovim"]
---

为什么 nvim?

快速安装

```bash
brew install nvim
```

插件管理

Packer 插件

```
$ mkdir -p ~/.local/share/nvim/site/pack/packer/start/packer.nvim
$ git clone --depth 1 https://github.com/wbthomason/packer.nvim ~/.local/share/nvim/site/pack/packer/start/packer.nvim
```

开始配置

引入 Packer

```bash
mkdir ~/.config/nvim/lua
nvim ~/.config/nvim/lua/plugins.lua
```

加载配置

```lua
return require('packer').startup(function(use)
  use 'wbthomason/packer.nvim'
end)
```
加载插件

plugins.lua

```lua
return require('packer').startup(function(use)
  use 'wbthomason/packer.nvim'
  use 'williamboman/mason.vim'
  use 'williamboman/mason-lspconfig.vim'
  use 'neovim/nvim-lspconfig'
end)
```

重载文件

```vim
:%luafile %
```

安装插件

```vim
:PackerInstall
```

nvim-lspconfig 只负责管理配置，不同语言的 Language Server 还独立安装。

配置 mason

配置 mason，使用 `:MasonInstal gopls`，依赖 golang 是否存在。

配置 gopls 与测试

We'll be leveraging hrsh7th plugin for this or plugins.

避免过多的消息


