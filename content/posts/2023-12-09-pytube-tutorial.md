---
title: "基于 Python 视频搬运 Part4 - pytube 下载 YouTube 资源"
date: 2023-12-09T14:06:15+08:00
draft: false
comment: true
description: "本文是 pytube 教程，介绍如何使用 pytube 实现通过 Python 下载油管音视频等资源。"
---

本文是基于 Python 视频搬运的第三篇，也是一篇完整的 pytube 教程，介绍如何通过 pytube 下载 YouTube 的音视频等资源。

## 概述

pytube 是一款由 Python 实现，用于下载油管的第三方库，它的特点是无第三方的依赖，轻量，而且还有一个非常重要的点，它的接口灵活度高。

而且，对于命令行，pytube 也提供了 pytube 命令也实现了通过命令行实现资源下载。当然，命令行下载视频的工具，更著名的 [youtube-dl](https://github.com/ytdl-org/youtube-dl) 和 [you-get](https://github.com/soimort/you-get)，它们比 pytube 出名，如果不是通过 Python 实现资源下载，它们或许是更好的选择。

## 安装

如下命令安装 pytube：

```bash
pip install pytube
```

## 快速开始

我们通过一个案例演示如何使用 pytube 下载视频，视频地址：www.youtube.com/watch?v=ceRYL271cao。

YoutTube 是 pytube 的核心类，它可用于获取某个 YouTube 视频的信息，包括基本属性内容等，如标题、音视频流，字幕等。

### 视频属性

```python
from pytube import YouTube

yt = YouTube("https://www.youtube.com/watch?v=ceRYL271cao")
print(f"title: {yt.title}\nthumbnail_url: {yt.thumbnail_url}\nchannel_url: {yt.channel_url}")
```

如上的代码获取了视频的标题、封面图和频道地址。

输出结果：

```bash
title: Be a tmux KING with Tmuxifier | My FAVORITE tmux tool
thumbnail_url: https://i.ytimg.com/vi/ceRYL271cao/hq720.jpg
channel_url: https://www.youtube.com/channel/UCo71RUe6DX4w-Vd47rFLXPg
```

更多属性：

```python
# dir 查看 YouTube 属性
dir(yt)
```

输出结果：


```bash

['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_age_restricted', '_author', '_embed_html', '_fmt_streams', '_initial_data', '_js', '_js_url', '_metadata', '_player_config_args', '_publish_date', '_title', '_vid_info', '_watch_html', 'age_restricted', 'allow_oauth_cache', 'author', 'bypass_age_gate', 'caption_tracks', 'captions', 'channel_id', 'channel_url', 'check_availability', 'description', 'embed_html', 'embed_url', 'fmt_streams', 'from_id', 'initial_data', 'js', 'js_url', 'keywords', 'length', 'metadata', 'publish_date', 'rating', 'register_on_complete_callback', 'register_on_progress_callback', 'stream_monostate', 'streaming_data', 'streams', 'thumbnail_url', 'title', 'use_oauth', 'vid_info', 'video_id', 'views', 'watch_html', 'watch_url']
```

属性的具体含义，可查看 pytube 文档：[YouTube Object](https://pytube.io/en/latest/api.html#youtube-object)

### 音视频流

pytube 与其他的油管视频下载工具不同，它不会默认选择资源下载，要我们手动 filter 选择要使用的资源。

首先，查看 YouTube 中可用的音视频资源信息，通过 YoutTube 的 streams 属性获取。

代码如下：

```
print(yt.streams)
```

输出结果：

```bash
[<Stream: itag="17" mime_type="video/3gpp" res="144p" fps="7fps" vcodec="mp4v.20.3" acodec="mp4a.40.2" progressive="True" type="video">,
<Stream: itag="18" mime_type="video/mp4" res="360p" fps="30fps" vcodec="avc1.42001E" acodec="mp4a.40.2" progressive="True" type="video">,
<Stream: itag="22" mime_type="video/mp4" res="720p" fps="30fps" vcodec="avc1.64001F" acodec="mp4a.40.2" progressive="True" type="video">, 
<Stream: itag="313" mime_type="video/webm" res="2160p" fps="30fps" vcodec="vp9" progressive="False" type="video">,
<Stream: itag="271" mime_type="video/webm" res="1440p" fps="30fps" vcodec="vp9" progressive="False" type="video">,
<Stream: itag="137" mime_type="video/mp4" res="1080p" fps="30fps" vcodec="avc1.640028" progressive="False" type="video">,
<Stream: itag="248" mime_type="video/webm" res="1080p" fps="30fps" vcodec="vp9" progressive="False" type="video">,
...
<Stream: itag="139" mime_type="audio/mp4" abr="48kbps" acodec="mp4a.40.5" progressive="False" type="audio">,
<Stream: itag="140" mime_type="audio/mp4" abr="128kbps" acodec="mp4a.40.2" progressive="False" type="audio">,
<Stream: itag="249" mime_type="audio/webm" abr="50kbps" acodec="opus" progressive="False" type="audio">,
<Stream: itag="250" mime_type="audio/webm" abr="70kbps" acodec="opus" progressive="False" type="audio">,
<Stream: itag="251" mime_type="audio/webm" abr="160kbps" acodec="opus" progressive="False" type="audio">]
```

pytube 支持 filter 过滤资源，如只要视频资源，选择分辨率最高的视频。

代码如下：

```python
stream = yt.streams.filter(type="video").order_by("resolution").last()
```

首先，通过 `type="video"` 过滤视频资源，接着按 resolution，即分辨率，升序排序视频，再最后一个视频。

### 视频下载

一旦获取到我们的目标 stream，就可以调用 download 方法，即可下载。

```python
stream.download()
```

默认情况，download 下载位置为当前目录，并以 YouTube 视频标题作为文件名。如希望修改这个默认行为，可通过 filename 和 output_path 实现。filename 执行下载文件名，output_path 执行文件下载路径。

## 音视频分离

现在，我们得到了一个视频文件，但这极有可能一个无声视频。

pytube 官方文档有如下这么一段话：

```bash
You may notice that some streams listed have both a video codec and audio codec, while others have just video or just audio, this is a result of YouTube supporting a streaming technique called Dynamic Adaptive Streaming over HTTP (DASH).

In the context of pytube, the implications are for the highest quality streams; you now need to download both the audio and video tracks and then post-process them with software like FFmpeg to merge them.

The legacy streams that contain the audio and video in a single file (referred to as “progressive download”) are still available, but only for resolutions 720p and below.
```

简单来说，就是部分 stream 同时包含音视频，而部分 stream 是音视频分离的。而同时包含视频的流使用的是以前的 Progressive Adaptive 技术，音视频分离使用的是 Dynamic Adaptive Streaming over HTTP，即 DASH 技术。

关键是，Progressive Adaptive 的视频最高分辨率是 720p，更高分辨率则必须是用 Dash 技术的流，分别下载音频和视频，再通过类似 FFmpeg 的工具将两者合并。

我们现在就不能只下载视频资源了。

修改后的代码，如下所示：

```python
dash_streams = yt.streams.filter(is_dash=True)
video_stream = dash_streams.filter(type="video").order_by("resolution").last()
audio_stream = dash_streams.filter(type="audio").first()

video_stream.download()
audio_stream.download()
```

到此，我们成功获取需要的音视频资源了。

### 下载进度

默认的配置，pytube 下载视频不展示下载进度，可通过在创建 YouTube 类时指定回调.

代码如下所示：

```python
from pytube import YouTube
from pytube.cli import on_progress

watch_url = "https://www.youtube.com/watch?v=ceRYL271cao"
yt = YouTube(watch_url, on_progress_call=on_progress)
```

除了 on_progress_callback，还可指定 on_complete_callback，用于指定下载完成的回调。

## 频道与播放列表

如何更好地使用 pytube 管理与下载资源呢？

YouTube 核心有 4 个 Object 类，即 Channel、PlayList、YoutTube 和 Stream。

前面已经演示过 YouTube 和 Stream 类的基本使用。要用好 pytube，重点是了解这 4 个类间的关系。

简言之，Channel 中包含 PlayList 和 YouTube。而 PlayList 中包含 YouTube。YouTube 中包含 Stream。即Channel 和 Playlist 是 YouTube 的容器。

如果实现频道视频检测的功能，可利用 Channel 和 PlayList 这两个类，定时检测 Channel 和 PlayList 中的 video_urls。

创建 Channel 的代码：

```python
from pytube import Channel

ch = Channel("https://www.youtube.com/channel/UCo71RUe6DX4w-Vd47rFLXPg")
```

频道的创建要传入 channel 地址，而这个地址，因为 YouTube 的改版，网页端中的 channel 地址不可用，而要通过 YoutTube 的 channel_url 属性获取。

示例代码如下：

```python
yt = YoutTube("https://www.youtube.com/watch?v=ceRYL271cao")
print(yt.channel_url)
```

输出结果：

```bash
https://www.youtube.com/channel/UCo71RUe6DX4w-Vd47rFLXPg
```

特别说明，因为 youtube 的改版，pytube 中有些接口已经不可用了，要修复。如，Channel 中的一些功能不可用，这篇文章有介绍修复方案：[解决近期Pytube的Channel Video列表会空的问题](https://zhuanlan.zhihu.com/p/586803092)。

通过 Channel 获取所有的 YouTube 或视频地址列表：

```python
print(ch.videos)
print(ch.video_urls)
```

播放列表 playlist 的创建，可通过播放 playlist 地址或视频地址中包含 playlist 的参数。

代码如下所示：

```python
from pytube import Playlist

p = Playlist("https://www.youtube.com/playlist?list=PLsz00TDipIfdGZRWNvOdeZfbka9HfpYBg")
# p = Playlist("https://www.youtube.com/watch?v=capyZ2D9Yz0&list=PLsz00TDipIfdGZRWNvOdeZfbka9HfpYBg&ab_channel=typecraft")
```

通过 playlist 获取所有 YoutTube 或地址列表：

```python
print(p.videos)
print(p.video_urls)
```

## 搜索能力

pytube 还提供了搜索油管的能力，或许一些场景会有用吧。

使用代码如下：

```python
from pytube import Search

results = Search("zsh").results
print(results)
```

其中返回的 results 就是 YouTube 的实例数组。如有需要，从中选择需要的 YoutTube 兑现个，在其上执行 streams 下载即可。

## 命令行工具 - pytube

pytube 除了以 python 库的形式提供视频下载能力，还提供了一个 pytube 的命令行工具。

以一些示例说明 pytube 命令的使用吧，如下所示：

下载包含音频且最高分辨率视频：

```bash
pytube https://www.youtube.com/watch?v=ceRYL271cao
```

查看所有可用的流：

```bash
pytube https://www.youtube.com/watch?v=ceRYL271cao --list
```

下载指定的流：

```bash
pytube https://www.youtube.com/watch?v=ceRYL271cao -itag=22
```

查看可用字幕：

```bash
pytube https://www.youtube.com/watch?v=ceRYL271cao -list-caption
```

下载执行字幕：

```bash
pytube https://www.youtube.com/watch?v=ceRYL271cao -c en
```

下载音频：

```bash
pytube https://www.youtube.com/watch?v=ceRYL271cao -a
```

## 最后

本文介绍了如何使用 pytube 下载油管资源，如果大家希望通过 python 管理下载视频，希望它能有所有帮助。

我的博文：[基于 Python 视频搬运 Part2 - pytube 下载 YouTube 资源](https://www.poloxue.com/posts/2023-12-07-pytube-tutorial/)。
