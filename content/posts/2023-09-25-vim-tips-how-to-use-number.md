---
title: "Vim 小技巧：高效利用 vim 的行号"
date: 2023-09-25T15:06:02+08:00
draft: false
comment: true
tags: ["vim", "neovim"]
---

我们知道，Vim 支持配置是否显示行号，对这个行号认知，我们一般指的是绝对行号。其实 Vim 支持配置两种行号模式：`number`（绝对行号） 和 `relativenumber`（相对行号）。

今天，基于 vim 行号介绍一个提升其使用效率的小技巧，混合使用 number 和 relativenumber。

## 绝对行号 number

绝对行号 `number`，我们基本都熟悉怎么使用。效果图如下所示： 

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-vim-tips-how-to-use-number-02.gif)

通过 `set number` 显示行号。默认开启的话，配置到 Vim 配置文件中即可。

其他命令：

```vim
" 显示行号
set nu " set number 的缩写形式
" 隐藏行号
set nonumber " 无缩写
set nonu     " 缩写形式
```

基于行号 `number`，实现的一些快捷操作，如：

- 基于行的快速跳转 `10G` 或 `:10`，快速跳转到第 10 行；
- 粘贴指定范围文本 `:10,20y` 或删除 `:10,20d`；
- 替换指定范围文本 `:10,20s/hello/world/g`；

> 注：`set numberwidth=4` 可配置行号所在的列的默认宽度为 4，如果行号数值达到 5 位数，将会自动扩展到 5 位。另外说明，不同于 Vim 的默认值是 2，Neovim 的默认宽度也是 4。

## 相对行号 relativenumber

在谈相对行号前，其实 Vim 另外一种行间跳转方式：基于相对位置，可使用如 `10k` 或 `10l` 向上向下快速跳转，是一种适合在屏幕范围快速跳转的方式。

但它缺点是，要数这个相对位置，如上图中，假设要从 `vim.o.shiftwidth = true`  跳转到 `vim.o.autoindent=true` ，可使用 `:13` 或者通过 `6l`（向下跳转 6 行）实现，第二种方式快速计算相对行号，使用 `13-7 = 6`，有一个计算过程，并不方便。

如何解决？

我们可引入 `relativenumber`，即相对行号，效果图如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-09-25-vim-tips-how-to-use-number-01.gif)

当前位置的相对行号是 `0`，而上下行号以此为基准递增，是动态变化的数值。基于相对行号，就能在通过相对位置跳转到窗口范围内任意一行。

相关配置命令：

```vim
set rnu " set relativenumber 的缩写形式"

" 隐藏相对行号
set norelativenumber
set nornu " set norelativenumber 的缩写形式
```

> 特别说明：如果 `number` 和 `relativenumber` 同时开启的情况下，当前所在行显示绝对行号，而其他行则显示相对行号。

## 按模式切换

有人提出一种将 `number` 和 `relativenumber` 结合使用的方式：在 `Insert` 模式下使用绝对行号，在其他模式下使用相对行号。猜测原因是，相对行号主要还是在非 `Insert` 模式下使用。

这样的效果就能同时兼顾 `number` 和 `relativenumber`。


配置 `vimrc`，语句如下：

```vim
augroup numbertoggle
  autocmd!
  autocmd BufEnter,FocusGained,InsertLeave,WinEnter * if &nu && mode() != "i" | set rnu   | endif
  autocmd BufLeave,FocusLost,InsertEnter,WinLeave   * if &nu                  | set nornu | endif
augroup END
```

或者

如果是 Neovim，引入插件：[nvim-numbertoggle](https://github.com/sitiom/nvim-numbertoggle) 亦可。

简洁的源码，如下所示：

```lua
local augroup = vim.api.nvim_create_augroup("numbertoggle", {})

vim.api.nvim_create_autocmd({ "BufEnter", "FocusGained", "InsertLeave", "CmdlineLeave", "WinEnter" }, {
   pattern = "*",
   group = augroup,
   callback = function()
      if vim.o.nu and vim.api.nvim_get_mode().mode ~= "i" then
         vim.opt.relativenumber = true
      end
   end,
})

vim.api.nvim_create_autocmd({ "BufLeave", "FocusLost", "InsertEnter", "CmdlineEnter", "WinLeave" }, {
   pattern = "*",
   group = augroup,
   callback = function()
      if vim.o.nu then
         vim.opt.relativenumber = false
         vim.cmd "redraw"
      end
   end,
})
```

注意下，这里加了个限制，只有在启动 `number` 的情况下，才会按模式切换 `relativenumber`。

## 最后

实话实说，虽然这方式看起来挺炫酷，但我一直不太习惯。毕竟，窗口范围内，我也可以使用绝对行号。或许是我没有养成良好的习惯。这次决定尝试下，开启这个配置，锻炼下自己的思维和手指。

博文地址：[Vim 小技巧：高效利用 vim 的行号](https://www.poloxue.com/posts/2023-09-25-vim-tips-how-to-use-number/)
