---
title: "使用 Whisper 和 MoviePy 实现视频自动加字幕"
date: 2024-01-05T21:39:29+08:00
draft: false
comment: true
description: "本博文将就此主题展开，介绍如何基于 openai-whisper 和 moviepy 实现通过 Python 快速给视频添加字幕。"
---

对于视频编辑而言，给视频添加字幕是一个常见的需求。而利用 python 的三方库 moviepy 和 openai-whisper，就可以轻松实现为视频添加字幕。

本博文将就此主题展开，介绍如何通过 Python 快速添加视频字幕。

## 前言概述

编辑视频时，为视频添加字幕不仅仅可以让内容易于理解性，而且可以增加视频的吸引力。

Python 提供了 MoviePy 库，同时近期开源的 openai-whisper 模块使得为视频添加字幕变得十分简单。

whisper 的安装直接通过如下命令即可完成。

```python
pip install openai-whisper 
```

而 moviepy 的安装可参考我之前的文章：[Python 视频剪辑库 - MoviePy 基础使用](https://www.poloxue.com/posts/2024-01-03-moviepy-basic-usage/)。

## 步骤介绍

添加字幕的核心步骤主要三步，如下所示：

- 利用 openai-whisper 从视频中将语音识别为文本；
- 基于识别结果生成 moviepy 字幕片段 SubtitlesClip；
- 将字幕片段 SubtitlesClip 和视频片段 VideoFileClip 合成最终文件；

让我们进入开始具体的操作。

## 语音识别

我们现在在当前目录有一个视频，名为 input_video2.mp4。可直接利用 whisper 的 transcribe 识别视频文件中的语音文本。

实现代码如下：

```python
import whisper

model = whisper.load("medium")
result = model.transcribe("input_video2.mp4")
print(json.dumps(result))
```

输出 result，如下所示
```bash
{
  "text": " In an era dominated by short-form videos, numerous user-friendly editing software options exist. However, for specialized video formats like e-book presentations or comic readings, streamlining the video creation process through automation stands as a crucial enhancement for efficiency.",
  "segments": [
    {
      "id": 0,
      "seek": 0,
      "start": 0.0,
      "end": 6.7,
      "text": " In an era dominated by short-form videos, numerous user-friendly editing software options exist.",
      "tokens": [],
      "temperature": 0.0,
      "avg_logprob": -0.170647520768015,
      "compression_ratio": 1.4947916666666667,
      "no_speech_prob": 0.007638249080628157
    },
    {
      "id": 1,
      "seek": 0,
      "start": 6.7,
      "end": 12.0,
      "text": " However, for specialized video formats like e-book presentations or comic readings,",
      "tokens": [],
      "temperature": 0.0,
      "avg_logprob": -0.170647520768015,
      "compression_ratio": 1.4947916666666667,
      "no_speech_prob": 0.007638249080628157
    },
    {
      "id": 2,
      "seek": 0,
      "start": 12.0,
      "end": 19.0,
      "text": " streamlining the video creation process through automation stands as a crucial enhancement for efficiency.",
      "tokens": [],
      "temperature": 0.0,
      "avg_logprob": -0.170647520768015,
      "compression_ratio": 1.4947916666666667,
      "no_speech_prob": 0.007638249080628157
    }
  ],
  "language": "en"
}
```

可以看到，result 中包含一个 segments 的数组，start 表示开始时间，end 表示结束时间，而 text 就是字幕文本。

好了，现在我们就有了制作字幕所需的所有信息。

## 字幕片段

接下来，让我们利用 whisper 的 result 生成字幕所需的文本片段 TextClip。

制作字幕片段一般方法是，循环遍历 segments 制作 clips 数组，使用 set_start 和 set_end 设置 TextClip 的开始结束时间。

```python
subtitle_clips = []
for segment in segments:
  subtitle_clip = TextClip(segment['text']).set_start(segment['start']).set_end(segment['end'])
  subtitle_clips.append(subtitle_clip)
```

moviepy 提供一个 moviepy.video.tools.subtitles.SubtitlesClip 的类。

我们只需将 segments 转为 `[((start, end), text)]` 的数组，传递给 SubtitlesClip，即可创建出 SubtitlesClip。

代码如下所示：

```python
from moviepy.video.tools.subtitles import SubtitlesClip

subtitles = []
for segment in segments:
  subtitle = (segment['start'], segment['end']), segment['text']
  subtitles.append(subtitle)

subtitles_clip = SubtitlesClip(subtitles)
```

OK，现在我们已经成功创建了字幕 clip 文件。

## 合成视频

有了字幕片段 Clip，让我们将其与原视频合成，生成一个包含字幕的文件。

代码如下所示：

```python
video = VideoFileClip("input_video2.mp4")
video = CompositeVideoClip([video, subtitles_clip])
video.write_videofile("output_with_subtitles.mp4", audio_codec="aac")
```

执行完代码，就可以生成带有字幕的视频文件。

效果如下所示：

{{< image "2024-01/2024-01-06-add-subtitles-using-moviepy-and-whipser-01.png" >}}

仔细观察左上角，可以看到字幕已经添加到视频上。

## 字幕位置

我们检查生成的视频文件，发现生成字幕位于视频的左上角，这个效果并不是我想要的。

如果想修改字幕在视频中的位置该如何做呢？

字幕正常应该是位于视频下方居中的。我们可以通过 `SubtitlesClip` 的方法 `set_position` 调整位置。

我们的目标是基于原视频，将字幕调整到位于视频底部上下 0.2 的位置，即坐标向下 0.8 的位置。

实现代码如下所示：

```python
width, height = video.size
pos = ("center", height * 0.8)
subtitles_clip = subtitles_clip.set_position(pos)
```

效果如下所示：

{{< image "2024-01/2024-01-06-add-subtitles-using-moviepy-and-whipser-02.png" >}}

字幕在视频底部。

## 字幕样式

如果发现字体太小或者想修改字体或颜色，我们可以覆盖创建的 `SubtitlesClip` 的第三个三个参数 `make_textclip`。

假设将默认字体大小改为 48，字体改为font 为 Arial，字体颜色改为 orange。

实现代码如下所示：

```python
def make_textclip(txt):
    return TextClip(txt, fontsize=48, font="Arial", color="orange")

subtitles_clip = SubtitlesClip(subtitles, make_textclip)
```

如果发现生成字幕超出了原视频范围，可设置 `TextClip` 的参数 size 限制字幕宽度，设置参数 `method` 为 caption 实现自动换行。

实现代码，如下所示，
```python
def make_textclip(txt):
    return TextClip(txt, fontsize=48, font="Arial", color="orange", size=(width * 0.8, None), method="caption")
```

字幕宽度为原始视频宽度的 0.8 倍，如果大于设置宽度则自动换成。

效果如下：

{{< image "2024-01/2024-01-06-add-subtitles-using-moviepy-and-whipser-03.png" >}}

到此，我们通过 openai-whisper 和 moviepy 制作自动化制作视频字幕的教程就完成了。 

## 总结

本博文主要介绍如何基于 openai-whisper 和 moviepy 实现自动化制作视频字幕。如果想了解字幕的更多样式，可查看 TextClip 还支持哪些参数。

还有，有了这个能力。如想继续扩展，我们还可以通过调用三方翻译软件，实现给视频自动加载翻译字幕。

感谢阅读，可持续关注我的博客。
