---
title: "终端环境：iTerm2 的安装、体验，还有自动换背景、自动管理布局"
date: "2023-09-28T19:23:22+08:00"
draft: false
comment: true
tags: ["zsh", "iterm2"]
description: "本系列的目标是介绍如何基于 iTerm2、zsh、Tmux 和 Neovim 搭建我的日常开发环境"
---

{{< video bb_id=619786097 yt_id=vNX_sReGZ5E >}}

我想大部分程序员在平时的工作中都是离不开终端，特别是如果你的系统是 MacOS 或 Linux 的话，终端的地位更是遥遥领先。

本系列的目标是介绍如何基于 iTerm2、zsh、oh-my-zsh（包括高效插件）、高效 shell 命令，甚至将计划基于 Tmux 和 Neovim 搭建我的日常开发环境。

本文是搭建我的开发环境系列中的第一篇，将介绍第一个必不可可少的工具终端 - iTerm2，Mac 系统上的终端神器。

## 前言介绍

我将主要介绍如何安装与配置 iTerm2，安装成功后，会带着一起体验的一些能力。

首先，iTerm2 是一款终端软件，它是 macOS 下默认终端 Terminal 的替代品。每次拿到新电脑，或者因某种原因重装系统，我首先要做的就是下载 iTerm2 来替换默认的终端 terminal。

## iTerm2 vs 默认 Terminal

为什么要用 iTerm2 替换默认的系统终端呢？这总要一些原因吧。

首先，iTerm2 相较于 Terminal 的优势就是，它更加美观，相对于默认终端，iTerm2 支持真彩，简单点说，你可以在终端显示图片，甚至是 gif 动图，无马赛克效果。

其他功能如分屏能力、颜色面板主题配置、搜索等肯定是基本能力，快捷键的定制性更强。另外，iTerm2，支持如 python 编程控制，可实现自动换背景效果。

还有，与 iTerm2 与 zsh 相结合体验更佳，zsh 部分会在后面慢慢介绍。

> 系列阅读：
>
> - [终端环境：iTerm2 的安装、体验与高级技巧](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [终端环境：高效 shell 命令（一）之目录文件命令 - exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...


废话不多说，接下来，让我们进入安装使用流程。

## 下载安装

安装可通过 [iTerm2 官网](https://iterm2.com/) 下载或者 MacOS 中 `brew` 安装，我将以 brew 安装为例。

如果还未安装 brew，安装命令：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装 iterm2，命令如下：

```
brew install --cask iterm2
```

等待执行完成，即安装完毕。

## 颜色面板主题

iTerm2 支持自定义颜色面板 color preset 设置。

首先，以 material design colors 为例，介绍如何安装设置。

安装命令，如下所示：

```bash
curl -Ls https://raw.githubusercontent.com/MartinSeeler/iterm2-material-design/master/material-design-colors.itermcolors > /tmp/material-design-colors.itermcolors && open /tmp/material-design-colors.itermcolors
```

如果成功执行，将会在 iTerm2 的 Preferences （使用快捷键 CMD+, 可快捷打开）-> Color 下的 "Color Presets" 中新增一条颜色面板选项，即 "material-design-color"。选中确认即可将 iTerm2 默认的颜色面板修改为 "material-design-color"。

上面的命令是通过 `curl` 下载配色文件，通过 `open` 打开直接安装。其实，稍微麻烦一点做法是，可通过设置面板 import 导入下载的后缀为 itermcolors 的文件。

如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-07-v1.png)

想有更多的选择，再安装两个配色方案：

安装 Snazzy：

```bash
curl -Ls https://raw.githubusercontent.com/sindresorhus/iterm2-snazzy/main/Snazzy.itermcolors > /tmp/Snazzy.itermcolors && open /tmp/Snazzy.itermcolors
```

安装 Dracula:

```bash
curl -Ls https://raw.githubusercontent.com/dracula/iterm/master/Dracula.itermcolors > /tmp/Dracula.itermcolors && open /tmp/Dracula.itermcolors
```
 
查看 Color Presets 面板，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-01.png)

选择一款你钟爱的配色，保存。

如果还没有满意的，可以从 [Iterm2-color-schemes](https://iterm2colorschemes.com/) 查找更多配色方案

## 如何使用

打开 'iTerm2'，快速使用体验一下吧。

### 分屏功能

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

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-05.jpeg)

> 注：图片取自 [Terminal vs iTerm2: Comparing Two CLI Tools on macOS](https://techwiser.com/terminal-vs-iterm2-comparison/#:~:text=1.-,Multiple%20Panes,panes%2C%20in%20the%20same%20window)

## 高级能力

这部分主要介绍 iTerm2 提供的 Python API，利用它，带你实现一些不一样的能力。

### 自动更换背景图

编程枯燥无味，想通过背景图为枯燥生活提供一些趣味。假设，我们要设计一个脚本，给终端一小时自动更好一个背景图。

假设我的壁纸图片都在 `~/Public/images/beauties` 目录下。

实现下这个代码，如下所示：

```python
import iterm2
import random
import os
import asyncio
import sys


# 替换背景图片的函数
async def change_background(session, image_path):
    # 获取当前会话的配置文件
    profile = await session.async_get_profile()
    # 设置背景图像位置
    await profile.async_set_background_image_location(image_path)


async def main(connection):
    app = await iterm2.async_get_app(connection)
    window = app.current_terminal_window

    if window is None:
        return

    session = window.current_tab.current_session

    # 获取脚本参数提供的目录
    directory = sys.argv[1] if len(sys.argv) > 1 else "."
    if not os.path.isdir(directory):
        raise ValueError(f"Provided path {directory} is not a directory")

    while True:  # 创建无限循环以定期更换背景
        # 获取目录中的所有图片文件
        image_files = [
            os.path.join(directory, f)
            for f in os.listdir(directory)
            if f.lower().endswith((".png", ".jpg", ".jpeg", ".tiff", ".bmp", ".gif"))
        ]

        if image_files:
            # 随机选择一张图片
            chosen_image = os.path.abspath(random.choice(image_files))
            # 调用函数更改背景
            await change_background(session, chosen_image)
        else:
            print(f"No images found in directory {directory}")

        # 等待一个小时
        await asyncio.sleep(3600)


iterm2.run_until_complete(main)
```

代码中提供了注释说明。

定时能力，我们是通过 `sleep(3600)` 实现每小时随机更换背景图片。特别说明，不要用 crontab，因为存在环境上下文问题，crontab 无法知道 iTerm2 的存在。

这个脚本可以放在 sh 的启动脚本中，如 `bashrc`、`zshrc` 等，这取决于你用什么 shell。我们只要将其设置为后台常驻的形式运行，

```bash
python random-bg.py your-images-directory &
```

### 布局自动化

如果，你用 tmux 的话，可能知道有布局管理工具是可帮你将布局自动创建的。其实，裸用 iTerm2 的情况下，同样可实现，因为 iTerm2 的 Python API 提供了分屏创建接口。

示例代码如下所示：

```python
import iterm2

async def main(connection):
    app = await iterm2.async_get_app(connection)
    window = app.current_terminal_window
    if window is not None:
        session = window.current_tab.current_session
        # 垂直分屏
        await session.async_split_pane(vertical=True)
        # 如果你想要水平分屏，将vertical参数设置为False
        # await session.async_split_pane(vertical=False)
    else:
        print("No current window")

iterm2.run_until_complete(main)
```

重点就是那句 `await session.async_split_pane(vertical=True)`，执行这个角度会自动进行左右分屏，即垂直分屏。

假设，你想自定义一些布局的话，如前面提到这个效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-02.jpeg)

我们来实现一个命令自动创建这个效果。

需求详细描述，iTerm2 自动创建三个 pane 的布局，左边 pane 用于通过 vim 命令打开指定目录（IDE 写代码)，右边上下两个 pane 中，上面的 pane 执行 `go run *.go` 命令，下面的 pane 创建清空等待输入。

编写这样一个脚本来实现这个流程，代码如下：

```python
import iterm2
import sys


async def main(connection):
    # 获取要打开的目录作为参数
    directory = sys.argv[1] if len(sys.argv) > 1 else "."

    app = await iterm2.async_get_app(connection)
    window = app.current_terminal_window

    if window is not None:
        # 创建左边的pane
        left_session = window.current_tab.current_session
        await left_session.async_send_text(f"cd {directory}\n")
        await left_session.async_send_text("vim\n")

        # 创建右边的pane
        # 右上的pane保持空白
        top_right_session = await left_session.async_split_pane(vertical=True)
        await top_right_session.async_send_text("\clear\n")
        await top_right_session.async_send_text("go run *.go\n")

        # 在右下的pane执行go run *.go
        bottom_right_session = await top_right_session.async_split_pane(vertical=False)
        await bottom_right_session.async_send_text("\clear\n")

    else:
        print("No current window")

iterm2.run_until_complete(main)
```

就不啰嗦代码逻辑了，非常简单。

命令效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-09.gif)

Python API 的使用就演示者两个案例。想了解它的更多能力，可直接查看它的官方文档，访问 [iTemr2 Python API Documentation](https://iterm2.com/python-api/)。

## 最后

本文重点介绍了 iTerm2 的安装、颜色面板的配置，还有体验其核心的能力，最有趣的部分是通过 iTerm2 的 Python API 能力，实现了一些自动化任务（自动更换背景和布局管理），提高趣味性和效率。

博文地址：[终端环境：终端环境：iTerm2 的安装与体验](2023-09-25-install-iterm2-as-my-developing-environment/)

