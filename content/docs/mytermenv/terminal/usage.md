---
title: "开始使用"
weight: 3
---

# 开始使用

打开 'iTerm2'，快速使用体验一下吧。

## 分屏功能

如下图所示，"CommandL+d" 垂直分屏，"Command+D" 水平分屏。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-02.jpeg)

当然，这个快捷键是可以配置的，因为这两个快捷键趋势不够便捷。我们打开它的快捷键配置，进入 Preferences -> Keys -> key Binds 即可开始设置快捷键键。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-08.png)

其中的红色框内容可用于设置如何水平和垂直分割屏幕：

- SHIFT+COMMAND+| -> 水平切片 split horizontally；
- SHIFT+COMMAND+- -> 水平切片 split horizontally；

其中的蓝色框区域可用于设置 vim 风格的上下左右分屏切换：

- COMMAND+h：向左选中
- COMMAND+j：向下选中
- COMMAND+k：向上选中
- COMMAND+l：向右选中

对于习惯使用 vim，但不是每个任务都有打开 tmux + vim 组合的时候，这个快捷键的设置就是效率神器啊。

### 分屏最大化

如下图所示，"Shift+Command+Enter" 分屏最大化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-03.jpeg)

再次 "Shift+Command+Enter" 恢复分屏。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-04.jpeg)

如果你习惯于在终端工作，那么分屏功能肯定是日常使用最多的能力了吧。如上的三分屏幕效果，左边座位代码编辑区域、右上方用于调试或运行代码，下面可用于其他一些测试或者运行命令区域。

### 支持搜索

相对于 通过 "Command+f" 开启搜索框：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-10.png)

iTerm2 的搜索能力更强大，可以在搜索框下拉仔细检查下它能力。诸如默认的 smartcase 模式、大小写敏感和不敏感模式、正则。可能你觉得这些不是很正常吗？俗话说，没有对比就没有伤害，如果你和系统默认的终端对比下就知道它的优秀之处了。

### 其他

iTerm2 是真彩 256 colors，这才让我们可以在 iTerm2 将 neovim 打造成媲美 vscode 的 IDE。

真彩测试，如下所示；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-12.png)

获取脚本，访问[脚本地址](https://github.com/gnachman/iTerm2/blob/master/tests/24-bit-color.sh)。

IDE 效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-14.png)

还有，我们可以直接在 iTerm2 查看图片，静态图片和 GIF 都是支持的。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-11.png)
![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-13.gif)

我现在就希望有一天终端支持内置浏览器，实现我 360 度无死角不用离开终端的梦想。我知道有一些文本 browser，如 w3m、lynx、links 等等。还有 browsh 这样的利用 firefox 渲染，终端展示的图形化支持的浏览器。但体验都巨差，无法真正意义上替换浏览器。

我的梦啊！

