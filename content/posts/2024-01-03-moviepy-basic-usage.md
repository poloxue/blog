---
title: "Python 视频剪辑库 - MoviePy 的基础使用"
date: 2024-01-02T17:41:45+08:00
draft: false
comment: true
description: "本视频将讨论的内容，介绍一个名为 MoviePy 的 Python 库，希望它能简化一些流程化的视频创作过程。本文是起始篇。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-03-moviepy-basic-usage-01.png)

短视频时代，有许多用户友好的剪辑软件可用。不过，对于某些模式比较固定的视频，如果能自动化简化视频制作对于提高效率实际很重要的。

这是本文将要讨论的内容。我们将利用一个名为 MoviePy 的 Python 库自动化您的视频制作过程。

本博文将探索 MoviePy，展示它如何通过一些简单的 Python 代码简化视频剪辑过程。

让我们开始吧。

## 安装

要使用 MoviePy，首先需要进行安装。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-03-moviepy-basic-usage-04-01.png)

首先是安装 moviepy 本身，我们可以使用 Python 的包管理器 - pip 完成这一步。

打开终端并执行如下命令：

```bash
pip install moviepy
```

必要的依赖项，如 ffmpeg，将会自动安装。

如果想创建文本片段，还需要 imagemagick。可以使用 HomeBrew 进行安装。执行命令：

```bash
brew install imagemagick
```

另一个重要的依赖是 pygame，特别适用于预览正在创建的视频。这很重要，因为生成新视频可能需要一些时间。如果您希望在制作过程中预览您的工作，您将需要安装 pygame。

当然！要安装 pygame，你可以在终端中执行:

```bash
pip install pygame
```

现在，MoviePy 的安装已经完成。让我们继续下一步 - 了解如何使用 MoviePy 编写代码。

## 导入文件

在 MoviePy 中，要访问所有必要的函数和类，只需输入：

```python
from moviepy.editor import *
```

要使用 MoviePy 操作视频，要通过 VideoFileClip 导入视频文件，代码如下：

```python
video = VideoFileClip("input_video.mp4")
```

## 查看属性

有了 `video` 这个 VideoFIleClip 实例，我们可以通过它查看视频的一些属性。

```python
print(f"video size: {video.size}") # 视频的大小分辨率
print(f"video duration: {video.duration}") # 视频的时间长度
print(f"video fps: {video.fps}") # 视频的每秒多少帧
```

## 调整大小

让我们从简单调整视频大小开始。

假设我现在有个视频，大小是 808x480，我的目标将其缩小 50%。只要一行代码轻松实现这个目标。

```python
video.resize(width=404, height=240)
```

可以看到我调用 `video.resize` 就将视频调整为 404x240 像素。

调用 `write_video_file` 方法，可将其保存视频文件，如为 "resized_video.mp4"。

```python
video.write_videofile("resized_video.mp4")
```

完成后，检查结果是否符合期望。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-03-moviepy-basic-usage-02.gif)

除了给出具体的长宽数值，如果是为了缩小 50%，还可以通过直接传递参数 0.5 实现。

```python
video = video.resize(0.5)
```

## 旋转

我们也可以使用 MoviePy 旋转视频。

编写代码如下：

```python
video = video.rotate(180)
```

这段代码将使用简单的 video.rotate 方法将视频旋转 180 度。我们将旋转后的版本保存为 "rotated_video.mp4"。

```python
video.write_videofile("rotated_video.mp4")
```

打开后会发现视频已经颠倒了。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-03-moviepy-basic-usage-03-01.gif)

## 片段剪辑

MoviePy 当然支持剪辑能力。

假设，我们想从长视频中剪辑出一个短视频非常简单，通过如下这段代码即可实现。

```python
video.subclip(5, 15).write_videofile("subclip_5_15_video.mp4")
```

我们使用 subclip 方法剪切从第 5 秒到第 15 秒的视频。我们将剪切版本保存为 "subclip_5_15_video.mp4"。

当然，ffmpeg 可以使用以下命令来完成，但显然要复杂些。

```bash
ffmpeg -i input_video.mp4 -ss 00:00:05 -t 10 -c:v copy -c:a copy subclip_5_15_video.mp4
```

## 导出 GIF

如果我们想从视频中创建一个 GIF，我们也可以做到！

例如，我们的目标是提取第 5 秒到第 6 秒视频片段，将其调整缩小 50% 并旋转 180 度，将结果保存为 GIF 图片。

代码如下所示：

```python
video.subclip(5, 6).resize(width=404).rotate(180).to_gif("output_video.gif")
```

经过 subclip resize 和 rotate 得到最终的 video，我们使用 to_gif 方法即可将其转换为名为 "output.gif" 的 GIF 文件。

效果如下所示：

## 视频合成

MoviePy 还支持视频合成，如现在有两个视频文件，分别是经过 resize 和 rotate 的视频。我们可直接将这两个视频合并顺序播放。

```python
video1 = video.resize(0.5)
video2 = video.rotate(90)

video = concatenate_video_clips([video1, video2])
video.write_videofile("output.mp4")
```

我们这里只是简单演示，视频合并的详细使用方法，后面会有文章介绍。

## 总结

本文简单展示了 MoviePy 视频处理的一些能力，没有详细展开介绍。如果你有兴趣进一步探索 MoviePy 的高级功能，请关注我的博客。

最后，让我们来看看 MoviePy 官方文档中的一些复杂示例。

- [Cup Song](https://www.youtube.com/watch?v=rIehsqqYFEM)
- [视频布局](https://www.youtube.com/watch?v=1hdgNxX-ta)
- [马赛克追踪](https://www.youtube.com/watch?v=zGhoZ4UBxEQ)
- [3D 效果](https://www.youtube.com/watch?v=M9R21SquDSk)

下篇文章：[MoviePy 中的音频操作](http://localhost:1313/posts/2024-01-03-moviepy-audio-file-clip/)
