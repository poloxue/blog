---
date: 2024-01-28
title: "Python API"
weight: 4
---

# Python API

这部分主要介绍 iTerm2 提供的 Python API，利用它，带你实现一些不一样的能力。我将演示两个案例，分别是背景图自动更换和分屏创建自动化。

## 环境准备

iTerm2 的 Python API 需要先启用脚本支持。打开 `Preferences > General > Magic > Enable Python API`。

安装依赖：

```bash
# 安装 iterm2 Python 包
pip install iterm2
```

iTerm2 的 Python API 通过 WebSocket 与 iTerm2 进程通信，脚本可以控制 iTerm2 的几乎所有方面——窗口、标签页、分屏、配色、文本内容等。

## 案例一：自动更换背景图

每天换个背景图保持新鲜感。这个脚本每天自动更换 iTerm2 的背景图片。

```python
#!/usr/bin/env python3
import iterm2
import random
from pathlib import Path

WALLPAPER_DIR = Path.home() / "Pictures" / "iterm2-wallpapers"

async def main(connection):
    # 获取所有 session
    app = await iterm2.async_get_app(connection)

    # 列出所有图片文件
    images = list(WALLPAPER_DIR.glob("*.{jpg,png,jpeg,webp}"))
    if not images:
        print(f"未在 {WALLPAPER_DIR} 中找到图片")
        return

    # 随机选一张
    wallpaper = random.choice(images)

    # 应用到所有 profile
    for profile in await iterm2Profile.async_query(connection):
        change = iterm2.LocalWriteOnlyProfile()
        change.set_background_image_location(str(wallpaper))
        await profile.async_set_local_profile(connection, change)

    print(f"背景图已更换为: {wallpaper.name}")

if __name__ == "__main__":
    iterm2.run_until_complete(main)
```

配合 cron 或 launchd 每天定时执行即可。

## 案例二：分屏创建自动化

如果你经常需要固定的开发布局——比如左边 2/3 编辑器，右边上下两个终端——可以写个脚本一键创建：

```python
#!/usr/bin/env python3
import iterm2

async def main(connection):
    app = await iterm2.async_get_app(connection)

    # 获取当前窗口
    window = app.current_terminal_window
    if window is None:
        print("没有打开的 iTerm2 窗口")
        return

    # 清除当前标签页的分屏
    session = window.current_tab.current_session

    # 垂直分屏（右边 1/3）
    right_session = await session.async_split_pane(
        vertical=True,
        before=False,
        profile="Default"
    )

    # 右边再水平分成上下两个
    await right_session.async_split_pane(
        vertical=False,
        before=False,
        profile="SSH"
    )

    # 左边打开编辑器
    await session.async_send_text("vim\n")

if __name__ == "__main__":
    iterm2.run_until_complete(main)
```

运行脚本：

```bash
python3 layout_dev.py
```

你就可以一键创建一个标准开发布局：左侧 Vim，右侧上下一分为二。

## 更多可能性

iTerm2 的 Python API 还能做很多事情：

- 根据当前 SSH 的目标服务器自动切换配色
- 定时轮换主题配色
- 监控终端活动并在指定条件时弹窗提醒
- 集成 CI/CD 流水线，在终端显示构建状态

文档参考：[iTerm2 Python API Documentation](https://iterm2.com/python-api/)

## 小结

iTerm2 的 Python API 为终端自动化提供了无限可能。从简单的背景图轮换到复杂的布局管理，只要你能想到的终端操作，基本上都能通过脚本实现自动化。
