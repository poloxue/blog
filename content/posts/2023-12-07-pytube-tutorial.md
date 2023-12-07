---
title: "基于 Python 视频搬运 Part2 - pytube 教程"
date: 2023-12-07T14:06:15+08:00
draft: true
comment: true
description: ""
---

本文是 pytube 教程，介绍如何使用 pytube 实现通过 Python 下载油管音视频等资源。

说明：pytube 是 Python 实现，用于下载的命令行工具和库，没有第三方的依赖，高度可靠。

安装：

```bash
pip install pytube
```

pytube 提供用于管理油管内容：

- 频道 - Channel；
- 播放列表 - Playlist；
- 视频 -> YouTube；

使用教程

基于代码

```
channel = Channel("")
playlist = Playlist("")
youtube = YouTube("")
```

命令行工具

pytube，是 pytube 提供的命令行工具。

```bash
pytube 
```

