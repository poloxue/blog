---
title: "Python API"
weight: 4
---

# Python API

这部分主要介绍 iTerm2 提供的 Python API，利用它，带你实现一些不一样的能力。我将演示两个案例，分别是背景图自动更换和分屏创建自动化。

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

    # 设置透明度，值在0（完全透明）到1（完全不透明）之间
    await profile.async_set_transparency(0.2)  # 设置为50%透明度
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

代码中提供了注释说明，不熟悉 Python 直接考虑即可，命令的接收参数是存放图片的目录。

定时能力，我们是通过 `sleep(3600)` 实现每小时随机更换背景图片。特别说明，不要用 crontab，因为存在环境上下文问题，crontab 无法知道 iTerm2 的存在。

这个脚本可以放在 sh 的启动脚本中，如 `bashrc`、`zshrc` 等，这取决于你用什么 shell。我们只要将其设置为后台常驻的形式运行，

```bash
python random-bg.py your-images-directory &
```

我假设将它设置为 1 秒换一次图片，效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-17.gif)

### 布局自动化

如果你用 tmux 的话，可能知道有些布局管理工具，如 tmuxifier 和 tmuxinator， 可自动创建布局。其实，只用 iTerm2，同样可实现这样的功能。

因为 iTerm2 的 Python API 提供了创建分屏的接口。

示例代码如下所示：

```python
import iterm2

async def main(connection):
    app = await iterm2.async_get_app(connection)
    window = app.current_terminal_window
    if window is None:
        print("No current window")
        return

    session = window.current_tab.current_session
    # 垂直分屏
    await session.async_split_pane(vertical=True)
    # 如果你想要水平分屏，将vertical参数设置为False
    # await session.async_split_pane(vertical=False)

iterm2.run_until_complete(main)
```

重点就是那句 `await session.async_split_pane(vertical=True)`，执行这个角度会自动进行左右分屏，即垂直分屏。

演示效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-09/2023-09-25-install-iterm2-as-my-developing-environment-16.gif)

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

Python API 的使用就演示这两个案例。想了解它的更多能力，可直接查看它的官方文档，访问 [iTemr2 Python API Documentation](https://iterm2.com/python-api/)。

