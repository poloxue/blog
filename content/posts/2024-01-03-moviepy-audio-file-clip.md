---
title: "Python 视频剪辑库 - MoviePy 的音频操作"
date: 2024-01-03T18:09:20+08:00
draft: false
comment: true
description: "本文将重点介绍 MoviePy 音频的一些操作，如音频提取创建、放大与片段剪辑等。"
---

上文介绍了如何安装 MoviePy 和利用 VideoFileClip 类读取视频文件，并进行一些简单的剪辑，如大小调整、旋转和片段剪辑等。

本文将重点介绍 MoviePy 音频的一些操作。

## 创建音频片段 AudioClip

MoviePy 中创建音频 Clip 的方法有两种，分别是从视频文件中提取音频，和基于音频文件创建。

### 基于视频文件提取音频片段

我们现在有一个视频文件，名为 `input_vidoe.mp4`。

直接通过如下代码提取音频文件，如下所示：

```python
from moviepy.editor import * 

video = VideoFileClip("input_video.mp4")
audio = video.audio
audio.write_audiofile("ouput_audio.mp3")
```

### 基于音频文件创建音频片段

通过 AudioFileClip 读取刚刚从视频中提取的音频文件即可。

代码如下所示：

```python
audio = AudioFileClip("output_audio.mp3")
```

我们可以将这个视频短片加载到另外一个视频片段中。

直接使用 TextClip 创建一个视频，我们通过 audio.duration 设置视频片段的播放时长。

```python
video = TextClip("Hello MoviePy", color="orange", size=(100, 100))
video = video.set_duration(audio.duration).set_fps(1)
```

让我们将音频加载到这个视频上。

代码如下：

```python
video.audio = audio
video.write_audio("text_video.mp4")
```


如果使用 mac 的 quicktimeplayer 打开，记得在写入的时候，加上参数 audio_codec="aac"，即如下所示：

```python
video.write_audio("text_video.mp4", audio_codec="aac")
```

完成之后，即可打开视频，检查下效果。

## 音频操作

介绍些 AudioClip 基础的操作，如音量的放大减小，音频片段裁剪等。

### 音量控制

音频片段的音量控制，直接通过 `AudioClip` 的 `volumex` 方法实现。

减低声音 0.5 倍的代码如下所示：

```python
audio = audio.volumex(0.5)
```

### 片段剪辑

现在，如果想剪辑音频中的一小段音频，和 VideoClip 一样，直接通过 subclip 实现即可。

如剪辑从 5 到 15 秒的音频片段，代码如下所示：

```python
audio = audio.subclip(5, 15)
```

### 删除视频中的音频

我们再介绍一个小技巧，如果希望删除一个视频中的音频，我们可以在通过 VideoFileClip 读取视频文件时，指定参数 `audio` 为 `False`。

```python
video = VideoFileClip("input_video", audio=False)
```

或者亦可通过调用 VideoFileClip 的 `without_audio` 方法删除视频片段中的音频。

```python
video = video.without_audio()
```

是不是非常简单。

## 总结

本文主要介绍了 MoviePy 中一些基础操作，更多关于 MoviePy 的教程，可订阅我的博客，感谢。

