---
title: "基于 Python 视频搬运 Part5 - 初始化与资源下载"
date: 2023-12-09T15:37:15+08:00
draft: true
comment: true
description: "本文介绍 mvideo 视频搬运项目的初始化与资源下载部分的实现代码。"
---

本文是基于 Python 视频搬运第四篇文章，介绍 mvideo 项目的初始化与资源下载的代码实现。

## 前言

我们在上文介绍了如何基于 pytube 实现 YouTube 资源的下载，包括视频音频，甚至是字幕的下载。当然，字幕在本项目中暂时不在本项目的计划之中。

本教程将会利用 pytube 这个 python 第三方包，我们将利用完成 YouTube 资源的下载。

## 主流程

在编写代码之前，初始化这部分的基本流程，如下所示：

{{< image "2023-12/2023-12-10-mvideo-init-function-01.png">}}

我将按照这三步流程，将相关的资源下载到特定位置。

## 查询可下载资源

我们将实现一个函数，用于从传递的参数 playlist 或 urls 到出可用的 YouTube 列表，从而从中获取 stream 下载资源。

```python
def extract_youtubes(urls, playlist, playlist_start, playlist_end):
    youtubes = []
    if playlist:
        playlist = Playlist(playlist)
        if playlist_start < playlist_end and playlist_end <= playlist.count:
          video_urls = playlist.video_urls[playlist_start:playlist_end]  # pyright: ignore
        else:
            youtubes = playlist.videos
    else:
        for url in urls:
            youtubes.append(YouTube(url, on_progress_callback=on_progress))

    return youtubes
```

