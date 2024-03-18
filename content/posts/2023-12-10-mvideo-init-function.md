---
title: "基于 Python 视频搬运 Part5 - 初始化与资源下载"
date: 2023-12-09T15:37:15+08:00
draft: false
comment: true
description: "本文介绍 mvideo 视频搬运项目的初始化与资源下载部分的实现代码。"
---

本文是基于 Python 视频搬运系列教程的第 5 篇，介绍 mvideo 项目的初始化与资源下载的实现。

## 前言

我们在上文介绍了如何基于 pytube 实现 YouTube 资源的下载，包括视频音频，甚至是字幕的下载。当然，字幕在本项目中暂时不在本项目的计划之中。

本教程将会利用 pytube 这个 python 第三方包，我们将利用完成 YouTube 资源的下载。

## 主流程

在编写代码之前，初始化这部分的基本流程，如下所示：

{{< image "2023-12/2023-12-10-mvideo-init-function-01.png">}}

我将按照这个流程，初始化项目，将资源下载到特定位置。

如下是主流程的实现代码：

```python
def init():
    env = Environment()

    try:
        # 1. Set translator and urls to config
        env.set_translator(translator, translator_from_lang, translator_to_lang)

        # 2. extract YouTubes from playlist or urls
        videos = extract_videos(urls, playlist, playlist_start, playlist_end)
        if not videos:
            raise RuntimeError("No videos found!")
        env.set_urls([url for url in videos])

        # 3. Download YouTubes
        for url, video in videos.items():
            output_path = env.add_video(
                url,
                video.title,
                video.description,
            )
            download_streams(env, video, output_path=output_path)
    except Exception:
        pass
    finally:
        # 4. Write config and data
        env.flush()
```

我们接下来的重点，只要完成 extract_videos 和 download_streams() 函数即可。

## 查询可下载资源

实现函数 extract_videos，用于从传递的参数 playlist 或 urls 查询可用的 YouTube 列表，从而从中获取 stream 下载资源。

```python
def extract_videos(urls, playlist, playlist_start, playlist_end):
    video_urls = None
    if playlist:
        playlist = Playlist(playlist)
        if playlist_start < playlist_end and playlist_end <= playlist.count:
            video_urls = playlist.video_urls[
                playlist_start:playlist_end
            ]  # pyright: ignore
        else:
            video_urls = playlist.video_urls
    else:
        video_urls = urls.split(",")

    videos = {}
    for url in video_urls:
        videos[url] = YouTube(url, on_progress_callback=on_progress)

    return videos
```

playlist 优先于 urls，如果 playlist 存在，从 playlist 查询资源。

## 下载音视频流

实现函数 download_streams 下载音视频资源，代码如下所示：

```python
def download_streams(env: Environment, video: YouTube, output_path: str):
    print("Downloading Video...")
    stream = (
        video.streams.filter(type="video", is_dash=True).order_by("resolution").last()
    )
    stream.download(  # pyright: ignore
        filename=env.video_filename, output_path=output_path
    )

    print("Downloading Audio...")
    stream = video.streams.filter(type="audio").first()
    stream.download(  # pyright: ignore
        filename=env.audio_filename, output_path=output_path
    )
```

到这里，init 功能初步完成，我们下面测试功能是否可用吧。

## 代码测试

假设现在我们地址为 www.youtube.com/watch?v=ceRYL271cao 的视频。

```bash
python mvideo/main.py init --urls "www.youtube.com/watch?v=ceRYL271cao"
```

执行完成后，会将在当前位置创建一个目录用于存放资源，以及配置文件 config.toml 和数据文件 data.json。

如下所示：

```bash
$ ls 
be-a-tmux-king-with-tmuxifier-|-my-favorite-tmux-tool  config.toml  data.json
```

在 be-a-tmux-king-with-tmuxifier-|-my-favorite-tmux-tool 目录中是下载的音视频文件。

```bash
$ ls
audio.mp4 video.webm
```

## 最后

本教程实现了 init 初始化子命令，初始化目录和下载资源。下篇教程开始介绍如何使用 whipser 语音识别生成字幕。

