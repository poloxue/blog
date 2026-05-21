---
title: "高级插件"
weight: 4
---

# 高级插件

我们再来了解 4 个非 oh-my-zsh 内置插件，它们 zsh-syntax-highlighting、zsh-autosuggestions、zsh-substring-history-search 和 you-should-use。这两个插件由 zsh 社区开发。

开始介绍前，先将这两个插件全部安装配置完成。

### 下载

下载命令如下所示：

```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
git clone https://github.com/MichaelAquilina/zsh-you-should-use.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/you-should-use
```

### 配置

打开 `.zshrc` 完成配置：
    
```zsh
plugins=(..., zsh-syntax-highlighting zsh-autosuggestions zsh-history-substring-search you-should-use))
```

记得执行 `source ~/.zshrc` 生效配置。

### 插件 1：zsh-syntax-highlighting

zsh-syntax-highlighting 是 zsh 的语法高亮插件，如果输入的命令不存在，或者输入 shell 语法不正确，将会自动以红色表示。它的优点就是，当我们在终端输入，实时输入实时反馈。

首先，我们尝试下错误命令，提示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-10.gif)

再来看看，正确命令提示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-11.gif)

对，就是这么简单。通过这个插件提供的实时反馈，可以防止我们在命令执行后，才知道输入错了。

### 插件 2：zsh-autosuggestions

zsh-autosuggestions 可以说是我最喜欢的插件了。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-16.gif)

它的作用是什么呢？

它可用于提示补全建议，当输入字符，默认情况下，它基于我们的历史命令自动提供输入建议。还记得前面提到的，zsh 的历史命令是在不同的会话间共享。现在，再结合 zsh-autosuggestions 插件，简直不要太爽。哈哈。

我们先看下效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-12.gif)

默认情况下，输入右方向键 → 可将建议直接输入终端。

但这个其实体验很差，对于一个双手不想离开键盘中心区域的人而言，通过右键接受提示建议，这简直不能忍啊。是否能改变这个默认快捷键呢？

我的目标是希望通过输入 `Ctrl + /` 接受建议，配置实现，如下所示：

```
# <Ctrl+/> 接受 auto-suggestion 的补全建议
bindkey '^_' autosuggest-accept
```

对！不要怀疑，CTRL+/ 的的字符表示就是 '^_'，我们可以通过执行 cat 命令查看，输入 CTRL+/，会到看如下输出。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-15-v1.gif)

如果你不知道想要设置的快捷键的字符表示，可以通过这种方式找到。

另外，如果希望 zsh-autosuggestion 不仅支持 history，也支持自动补全的建议提示，即原来那些要输入 tab 才能出现的内容，如子命令、命令选项、目录文件等提示，也能在提示建议的范围中。我们只需增加 completeion 这个配置项。

如下所示：

```zsh
export ZSH_AUTOSUGGEST_STRATEGY=(history completion)
```

现在，如果输入时，还没有历史命令可作为建议，会提供类似于目录、参数选项等建议。

类似于如下的效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-22.gif)


## 插件 3：zsh-history-substring-search

先介绍 [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)。它的主要用途是什么？

一般情况下，在使用 zsh 时，通过 ↑ 或 ↓ 方向键，能实现类似按前缀匹配补齐的效果。

而如果输入的是中间的字符串，则没法自动补齐。这个插件真是为这个目的而生的。

使用这个插件前，除了启用插件以外，还需要进一步配置下，将 zsh-history-substring-search 提供的能力绑定到快捷按键。

例如，上下方向键 ↑ 和 ↓。

```zsh
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
```

在生效配置后，测试失败的话，查看文档，其中有介绍：

> However, if the observed values don't work, you can try using terminfo:
>
> bindkey "$terminfo[kcuu1]" history-substring-search-up
> bindkey "$terminfo[kcud1]" history-substring-search-down

那我们就增加这两行配置吧。

```zsh
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down
```

除了 ↑ ↓ 按键外，我一般还习惯使用 CTRL+P/N 上下查找历史记录，配置如下：

```zsh
bindkey '^p' history-substring-search-up
bindkey '^n' history-substring-search-down
```

如果希望支持 vi 的 jk，配置如下：

```zsh
bindkey -M vicmd 'k' history-substring-search-up
bindkey -M vicmd 'j' history-substring-search-up
```

保存生效配置，测试下最终的成功成果吧。效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-04.gif)

另外，高亮样色可配置化的，可通过类似如下语法实现：

```
export HISTORY_SUBSTRING_SEARCH_HIGHLIGHT_FOUND=(bg=none,fg=magenta,bold)
```

设置 background 为 none，即无色，而 front 设置为 magenta,bold。效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-07.png)

如上的 zsh 的颜色变量，可查看 [zsh 仓库文档](https://github.com/zsh-users/zsh/blob/master/Functions/Misc/colors) 发现更多颜色。

```zsh
color=(
# Codes listed in this array are from ECMA-48, Section 8.3.117, p. 61.
# Those that are commented out are not widely supported or aren't closely
# enough related to color manipulation, but are included for completeness.

# Attribute codes:
  00 none                 # 20 gothic
  01 bold                 # 21 double-underline
  02 faint                  22 normal
  03 italic                 23 no-italic         # no-gothic
  04 underline              24 no-underline
  05 blink                  25 no-blink
# 06 fast-blink           # 26 proportional
  07 reverse                27 no-reverse
# 07 standout               27 no-standout
  08 conceal                28 no-conceal
# 09 strikethrough        # 29 no-strikethrough

# ...

# Bright color codes (xterm extension)
  90 bright-gray            100 bg-bright-gray
  91 bright-red             101 bg-bright-red
  92 bright-green           102 bg-bright-green
  93 bright-yellow          103 bg-bright-yellow
  94 bright-blue            104 bg-bright-blue
  95 bright-magenta         105 bg-bright-magenta
  96 bright-cyan            106 bg-bright-cyan
  97 bright-white           107 bg-bright-white
)
```

## 插件 4：you-should-use

[you-should-use](https://github.com/MichaelAquilina/zsh-you-should-use) 用途是，如果执行的命令存在别名，会自动提示推荐使用的别名；

由于，默认的提示信息在命令输出之前，添加如下配置：

```bash
export YSU_MESSAGE_POSITION="after"
```

它的作用是，实现将提示信息打印在命令输出的最后。

最终效果演示，如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-05.gif)

## 总结

本文介绍了 4 个 zsh 高级插件，每个插件都有特定的场景用途，希望能给大家的日常工作提升效率。


