---
title: "Python 视频剪辑库 - MoviePy 中 TextClip"
date: 2024-01-03T20:55:53+08:00
draft: false
comment: true
description: "本文将介绍 MoviePy 第三种基础 Clip - TextClip。"
---

前面的两篇教程，已经介绍了 MoviePy 的 VideoFileClip 和 AudioFileClip 的使用。本文将介绍 MoviePy 第三种基础 Clip - TextClip。

## 前言

什么是 TextClip？TextClip，即是用于生成基于文本视频片段的类。

TextClip 创建的包含文本的视频片段，一方面，我们可以将其作为单独的片段保存。另一方面，我们可以将其合成到其他视频中，这对于制作视频标题、说明，甚至字幕等内容非常有用。

关于字幕制作，会有单独一篇博文，如介绍何基于 MoviePy 和 Whisper 实现自动给视频添加字幕。

我们开始进入正题吧。

## 快速上手

让我们直接创建一个 TextClip。

示例代码，如下所示：

```python
# 创建一个包含文本的视频片段
text = "Hello MoviePy!"
txt_clip = TextClip(text, color='orange', size=(100, 100))

# 为视频片段设置持续时间
video_duration = 10  # 设置视频持续时间为 10 秒
txt_clip = txt_clip.set_duration(video_duration).set_fps(1)

# 保存生成的文本视频片段
txt_clip.write_videofile("text_clip.mp4")
```

这个例子展示了如何使用 TextClip 类创建一个文本视频片段。

代码具体作用是，创建了一个持续时间为 10 秒的视频片段，其中包含 "Hello MoviePy!" 文本，颜色是 orange，字体是 AvantGrade-Book，视频大小 (100,100)。

我们可以按需调整文本内容、字体，视频大小、颜色等，以及通过 set_duration 方法设置视频时长。

注意一点，对于 TextClip，如果希望保存为视频，则必须指定 fps。我们接下来将要介绍的 ImageClip 也是如此。

## 添加到视频

让我们把生成文本视频片段合成到现有的一个视频上。

代码如下所示：

```python
video = VideoFileClip("input_video.mp4")
video = CompositeVideoClips([video, txt_clip.set_duration(video.duration)])
video.write_videofile("output_video_with_text.mp4")
```

我们使用 CompositeVideoClips 类将 video 与 txt_clip 和合并成一个视频。

通过 set_pos 方法，我们还可以指定文本居中显示，代码如下：

```python
txt_clip.set_pos("center")
```

## 文本字体

MoviePy  通过将系统字体内置，让我们可以直接在 TextClip 中使用内置字体名称设置字体，而无需指定字体的具体路径。

```python
txt_clip = TextClip(text, color="orange", size=(100, 100), font="AvantGarde-Book")
```

如果查看 TextClip 支持的内置字体，可使用 TextClip.list('font') 列出所有支持的字体。

示例代码，如下所示：

```python
builtin_fonts = txt_clip.list('font')

# 打印内置字体列表
print("Built-in Fonts in TextClip:")
print(builtin_fonts)
```

输出内容：

```bash
[
  'AvantGarde-Book',
  'AvantGarde-BookOblique',
  'AvantGarde-Demi',
  ...
  'Yuppy-TC-Regular',
  'Zapf-Dingbats',
  'Zapfino'
]
```

例如，我们可以使用 Nerd Hack 字体生成包含 Icon 的文本内容。

文本内容：

{{< image "2024-01/2024-01-04-moviepy-text-clip-01.png" >}}

实现代码：

```python
txt_clip = TextClip(
  text,
  color='white',
  font="MesloLGS-NF-Regular",
  fontsize=36,
  size=(1920, 1080),
)
txt_clip = txt_clip.set_duration(10).set_fps(1)
txt_clip.write_videofile("text_video.mp4")
```

## 自定义字体路径

如果想在 TextClip 中使用特定字体，需要在系统中安装字体，或者代码中指定字体路径。

假设，我现在有一个字体，可以直接指定字体自定义路径。

示例代码如下：

```python
# 指定自定义字体的路径
custom_font_path = "/Users/jian.xue/Library/Fonts/MesloLGS NF Regular.ttf"

# 创建一个包含文本的视频片段，并指定字体路径
text = "Hello MoviePy!"
txt_clip = TextClip(text, fontsize=70, color='white', font=custom_font_path)
```

## TextClip 的底层

TextClip 的底层使用的其实是 imagemagick，可直接通过 magick 命令列出它的所有字体。

例如，查看支持所有字体，也可以通过如下命令。

```bash
magick -list font
```

输出如下：

```bash
...
  Font: MesloLGS-NF-Bold
    family: MesloLGS NF
    glyphs: /Users/jian.xue/Library/Fonts/MesloLGS NF Bold.ttf
  Font: MesloLGS-NF-Bold-Italic
    family: MesloLGS NF
    glyphs: /Users/jian.xue/Library/Fonts/MesloLGS NF Bold Italic.ttf
  Font: MesloLGS-NF-Italic
    family: MesloLGS NF
    glyphs: /Users/jian.xue/Library/Fonts/MesloLGS NF Italic.ttf
  Font: MesloLGS-NF-Regular
    family: MesloLGS NF
    glyphs: /Users/jian.xue/Library/Fonts/MesloLGS NF Regular.ttf
...
```

## 总结

本文介绍了如何使用 TextClip 类创建包含特定文本的片段。我们可以调整文本内容、尺寸和视频持续时间等参数来生成不同的视频片段，创造出更多多样且独特的视频内容。

如果觉得内容不错，可关注我的频道。
