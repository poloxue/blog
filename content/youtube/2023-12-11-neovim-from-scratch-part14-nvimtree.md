---
title: "从零开始 NeoVim IDE Part14 - 文件浏览 nvimtree"
date: 2023-12-08T19:22:21+08:00
draft: true
comment: true
description: "本视频介绍 nvim 插件 nvim-tree 的使用与配置。nvim-tree 是 neovim 文件资源管理器，使用 lua 实现，功能强大。"
---

本视频介绍 nvim 插件 [nvim-tree](https://github.com/nvim-tree/nvim-tree.lua) 的使用与配置。

视频来源 

> 这个视频教程是 nvim 从零开始系列的第 14 部分，介绍 neovim 文件树浏览器的使用。

nvim-tree 插件的仓库地址，访问 [nvim-tree](https://github.com/nvim-tree/nvim-tree.lua)，本视频对应 branch 为 12-nvimtree。

## 前言

[nvim-tree](https://github.com/nvim-tree/nvim-tree.lua) 是 neovim 文件资源管理器，功能强大，完全使用 lua 实现。

相对于 telescope，具有针对性的搜索，要提前对项目有一定的了解，直到想要寻找的文件。nvimtree 可显示项目的文件树，方便了解项目的文件目录是如何组织的，甚至是项目的设计架构。在其中，我们能遍历查看所有的文件。

{{< image "2023-12/2023-12-11-neovim-from-scratch-part14-nvimtree-01.png" >}}

nvimtree 的显示效果，如上图所示。

## 安装

本视频中，nvim 的插件是通过 packer 实现的。

使用 packer.nvim 安装：

```lua
packer.startup(function(use) 
  use 'kyazdani42/nvim-web-devicons'
  use 'kyazdani42/nvim-tree.lua'
end
```

这里引入了两个插件，[nvim-web-devicons](https://github.com/nvim-tree/nvim-web-devicons) 插件是为了让 nvim-tree 支持显示 filetype 图标。具体效果可查看上图。

## 配置

通过 packer 引入插件后，接着就是配置插件。插件的配置代码查看 [nvim-tree.lua]()

