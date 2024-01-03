---
title: "Python 视频剪辑库 - MoviePy 的基础使用"
date: 2024-01-03T17:41:45+08:00
draft: false
comment: true
description: "短视频时代，有许多用户友好的剪辑软件可选择。而对于某些专门的视频，如电子书演示或漫画朗读，自动化简化视频制作对于提高效率至关重要。这是本视频将讨论的内容。我们将利用一个名为 MoviePy 的 Python 库自动化您的视频制作过程。"
---

短视频时代，有许多用户友好的剪辑软件可选择。而对于某些专门的视频，如电子书演示或漫画朗读，自动化简化视频制作对于提高效率至关重要。

这是本视频将讨论的内容。我们将利用一个名为 MoviePy 的 Python 库自动化您的视频制作过程。

本教程中，我们将探索 MoviePy 的功能，展示它如何通过一些简单的 Python 代码简化视频剪辑。

让我们开始吧。

## 安装

要使用 MoviePy，首先需要进行安装。

我们可以使用 Python 的包管理器 - pip 来完成这一步。让我们打开终端并执行 pip install moviepy。

```bash
pip install moviepy
```

必要的依赖项，如 ffmpeg，将会自动安装。

如果你想要创建文本片段，你将需要 imagemagick。你可以轻松使用 HomeBrew 进行安装。执行命令：

```bash
brew install imagemagick
```

另一个重要的依赖是 pygame，特别适用于预览正在创建的视频。这很重要，因为生成新视频可能需要一些时间。如果您希望在制作过程中预览您的工作，您将需要安装 pygame。

当然！要安装 pygame，你可以在终端中执行:

```bash
pip install pygame
```

现在，MoviePy 的安装已经完成。让我们继续下一步 - 了解如何使用 MoviePy 编写代码。

## 示例

当然！为了演示如何使用 MoviePy，让我们创建一个文件，展示示例。

在 MoviePy 中，要访问所有必要的函数和类，你只需输入：

```python
from moviepy.editor import *
```

### 调整大小

我当前目录中有一个大小为 808x440 的视频文件。可以使用 VideoFileClip 大小属性检查尺寸。

```python
video = VideoFileClip("input_video.mp4")
print(video.size)
```

让我们从简单调整视频大小开始。

我们可以用一行代码轻松实现这个目标。

```python
video.resize(width=404, height=240)
```

你可以看到我调用 video.resize 就将视频调整为 404x240 像素。

将其保存为 "resized_video.mp4"。调用 write video file 方法。
```python
video.write_videofile("resized_video.mp4")
```

完成之后，可检查结果是否符合期望。

### 旋转

我们也可以使用 MoviePy 旋转视频。

编写代码如下：

```python
video = video.rotate(180)
```

这段代码将使用简单的 video.rotate 方法将视频旋转 180 度。我们将旋转后的版本保存为 "rotated_video.mp4"。

```python
video.write_videofile("rotated_video.mp4")
```

打开旋转后的视频会发现视频已经颠倒了。

### 片段剪辑

使用 MoviePy，从长视频中剪切出一个短视频非常简单。

写一段代码。我们使用 subclip 方法剪切从第 5 秒到第 15 秒的视频。

```python
video.subclip(5, 15).write_videofile("subclip_5_15_video.mp4")
```

然后将剪切版本保存为 "subclip_5_15_video.mp4"。

当然，ffmpeg 可以使用以下命令来完成，但显然要复杂些。

```bash
ffmpeg -i input_video.mp4 -ss 00:00:05 -t 10 -c:v copy -c:a copy subclip_5_15_video.mp4
```

## 导出 GIF

如果我们想从视频中创建一个 GIF，我们也可以做到！

例如，让我们使用 MoviePy 从视频段创建一个 GIF：

```python
video.subclip(5, 15).resize(width=404).rotate(180).to_gif("output_video.gif")
```

我们从第 5 秒到第 15 秒提取视频片段，然后将其调整为 404 像素宽度并旋转 180 度。使用 to_gif 方法将其转换为名为 "output.gif" 的 GIF。

最后，让我们来看看 MoviePy 官方文档中的一些复杂示例。

- [视频链接1](https://www.youtube.com/watch?v=rIehsqqYFEM)
- [视频链接2](https://www.youtube.com/watch?v=1hdgNxX-ta)
- [视频链接3](https://www.youtube.com/watch?v=zGhoZ4UBxEQ)
- [视频链接4](https://www.youtube.com/watch?v=M9R21SquDSk)

好了，以上就是今天的视频内容，我们涵盖了 MoviePy 的一些简单示例。

如果你有兴趣进一步探索 MoviePy 的高级功能，请关注我的博客。
