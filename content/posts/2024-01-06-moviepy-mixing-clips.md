---
title: "Python 视频剪辑库 - Moviepy 的混合剪辑 mixing clips"
date: 2024-01-06T20:35:11+08:00
draft: true
comment: true
description: "今天我们来介绍 MoviePy 的混合剪辑 - mixing clips。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-06-moviepy-mix-clips-07.png)

今天，介绍下 MoviePy 的混合剪辑 - mixing clips，即如何将多个 clip 合并为一个 clip 生成最终视频。

## 前言概述

混合剪辑，主要指的是如何将多个 Clip 合成为一个 Clip。Clip 的类型可以是 VideoClip、或者是 AudioClip。

MoviePy 中的混合剪辑 mixing clips 提供的方法，我大致上分三类，即是基于时间的 concantenate_clips、基于空间 clips_array 和随意合成 CompositeClips。不过，因为 audio 音频没有空间的概念，clips_array 只适用于 VideoClip 对象。

Ok, 那让我们具体介绍具体的使用吧。

## 基于时间 - concatenate clips

首先，我们介绍 concatenate clips 基于时间的视频合成。所谓，基于时间，即按时间依次播放片段。

### 视频合成

假设，我们要制作一个视频，依次播放 "1st second"、"2nd second" 和 "3rd second"。我们可以通过 `TextClip` 依次创建三个视频片段，然后通过 concantenate_video_clip 合并。

实现代码如下所示：

```python
from moviepy.editor import *

clip1 = TextClip("1st second", color="white", size=(1280, 720)).set_duration(1).set_fps(1)
clip2 = TextClip("2nd second", color="white", size=(1280, 720)).set_duration(1).set_fps(1)
clip3 = TextClip("3rd second", color="white", size=(1280, 720)).set_duration(1).set_fps(1)

video = concatenate_videoclips([clip1, clip2, clip3])
video.write_videofile("concatenate_videoclips.mp4")
```

视频如下所示，按时间依次播放片段。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-06-moviepy-mix-clips-02.gif)

### 音频合成

如果希望基于时间合成音频，直接使用 concantenate_audio_clips 方法，如假设我当前有 "1st-second.wav", "2nd-second.wav", "3rd-second.wav" 音频文件。

如下代码即可将代码按时间合成。
```python
clip1 = AudioFileClip("1st-second.wav")
clip2 = AudioFileClip("2nd-second.wav")
clip3 = AudioFileClip("3rd-second.wav")

audio = concatenate_audioclips([clip1, clip2, clip3])
audio.write_audiofile("concatenate_audioclips.wav")
```

最终生成的音频文件效果，将依次朗读 "1st second"，"2nd second"、"3rd second"。

## 基于空间 - clips_array

如果想让多个 Clip 在同一时刻显示且平铺开，直接使用 clips_array，按空间平铺多个 clip。

> 说明：音频不存在空间概念，故而 clips_array 只适用于操作视频。

我们还是使用 TextClip 创建 Clip，然后使用 clips_array 按数组拼接起来。

实现代码，如下所示：
```python
from moviepy.editor import *

clip1 = TextClip("1st second", color="white", bg_color="gray", size=(1280, 720)).set_duration(1).set_fps(1)
clip2 = TextClip("2nd second", color="white", bg_color="gray", size=(1280, 720)).set_duration(1).set_fps(1)
clip3 = TextClip("3rd second", color="white", bg_color="gray", size=(1280, 720)).set_duration(1).set_fps(1)
clip4 = TextClip("4th second", color="white", bg_color="gray", size=(1280, 720)).set_duration(1).set_fps(1)


clip1 = clip1.margin(10)
clip2 = clip2.margin(10)
clip3 = clip3.margin(10)
clip4 = clip4.margin(10)

video = clips_array([[clip1, clip2], [clip3, clip4]])
video.write_videofile("clips_array.mp4")
```

clips_array 的参数是一个二维数组，假设有一个 2 x 2 的数组，位置是：

```bash
[
  [行1列1, 行1列2],
  [行2列1, 行2列2]
]
```

我们最终合成视频的效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-06-moviepy-mix-clips-03.gif)

## 自由合成 - composite clips

无论是基于空间，还是基于时间，都是基于一个固定的模式合成。如果不希望局限于这种模式，就要使用 Composite Clips 类，自由合成。

### 视频合成

我们使用 moviepy 的 CompositeVideoClips 类可随意合成视频。

自由合成多个 Clip 的话，即某个 Clip 出现的时间和空间和其他 Clip 无关，意味我们要单独定义视频位置与播放的时间区域。

#### 设置位置

如何设置视频 Clip 位置呢？

VideoClip 提供了一个 set_position 方法，用它即可设置视频的位置，参数是 x y 坐标点。

坐标系如何确认呢？

使用 ConcompositeVideoClip 合成的 Clip，大小 size 默认是参数 clips 数组的第一个 Clip 的 size，而位置的 x 与 y 便是基于这第一个 Clip 而定的。

坐标系的原点位于左上角的位置，x 轴的方式从左向右，而 y 轴的方向是从上到下，如下图所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-06-moviepy-mix-clips-04.jpeg)

现在，我们尝试下，目标是将一个 TextClip 添加一个视频上，居中显示视频标题。

实现代码，如下所示：

```python
title = "MoviePy Basic Usage"
video = VideoFileClip("input_video2.mp4")
txt_clip = TextClip(title, fontsize=80, color="orange").set_duration(video.duration)

# 计算居中坐标位置
width, height = video.size
txt_width, txt_height = txt_clip.size
x = (width - txt_width) /2
y = (height - txt_height) / 2
position = (x, y)
txt_clip = txt_clip.set_position(position)

video = CompositeVideoClip([video, txt_clip])
video.write_videofile("CompositeVideoClip.mp4")
```

最终效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-06-moviepy-mix-clips-05.png)

示例代码中，有一串代码用于计算居中的位置。对于常见的对齐方式，如居中，在 moviepy 可直接使用 "center" 指定，其他 left，right, top, bottom 也是如此。

可通过如下代码直接将 txt_clip 居中。

```python
txt_clip = txt_clip.set_position(("center", "center"))
```

#### 设置出现时间段

本段介绍下如何配置 Clip 的显示时间，其核心是三个函数，`set_start`、`set_end` 和 `set_duration`，分别用于设置 Clip 的开始时间、结束时间和时间长度。

一个案例，假设我们想在视频添加另一个片段，依次显示 "3"、"2"、"1"，实现倒计时的效果。

```python
from moviepy.editor import *

video = VideoFileClip("input_video2.mp4")
txt_clips = []
start = 0
duration = 1
for text in range(3, 0, -1):
    txt_clip = TextClip(f"{text}", color="white", fontsize=128).set_start(start).set_duration(duration).set_fps(1)
    txt_clip = txt_clip.set_position(("center", "center"))
    txt_clips.append(txt_clip)
    start += duration

video = CompositeVideoClip([video] + txt_clips)
```

通过 for 循环依次生成 TextClip 并设置每个 Clip 的显示时间段。

效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-06-moviepy-mix-clips-06.gif)

### 音频合成

我们使用 CompositeAudioClip 类合成音频。常见的场景是，我们想在原视频的基础上，添加背景音乐。

由于，音频不存在位置的问题，就简单了很多，我们主要关注如何时间设置即可。

一个案例，我们有三个音频文件，分别 1st-second.wav、2nd-second.wav 和 3rd-second.wav，我们使用 concatenate_videoclips 依次播放，且在 1-5 秒加上背景因为。 

演示代码如下所示：

```python
from moviepy.editor import *

clip1 = AudioFileClip("./1st-second.wav")
clip2 = AudioFileClip("./2nd-second.wav")
clip3 = AudioFileClip("./3rd-second.wav")

bg_music = AudioFileClip("./bg-music.mp3")

clip1 = clip1.set_start(0).set_duration(1)
clip2 = clip2.set_start(1).set_duration(1)
clip3 = clip3.set_start(2).set_duration(1)
bg_music = bg_music.set_start(1).set_duration(4)

clips = [clip1, clip2, clip3, bg_music]
audio = CompositeAudioClip(clips).set_fps(44100)

audio.write_audiofile("CompositeAudioClip.mp3")
```

如此，我们就生成了一个带有背景音乐的音频文件。

## 特别说明

Clip 的合成要在相同类型 Clip 上进行，怎么理解？

简单来说，假设要在视频上添加音频，直接通过 set_audio 添加，视频上添加蒙层 mask 使用 set_mask 添加即可，而不是使用 concantenate_clips 或者 CompositeClips 实现。

## 总结

本博文详细介绍了 MoviePy 的各种混合剪辑方法，这块内容算是 MoviePy 中比较难理解的内容了，希望帮助大家更好地理解和使用这些功能。


