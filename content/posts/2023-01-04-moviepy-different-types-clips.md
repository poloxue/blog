---
title: "Python 视频剪辑库 - MoviePy 的不同 Clip 类"
date: 2024-01-03T17:55:53+08:00
draft: true
comment: true
description: ""
---


前文中，我已经介绍了 MoviePy 的安装与快速上手，核心用了 VideoFileClip 这个类，它读取视频文件，创建可供操作的视频实例。我们已经了解如何进行视频分辨率大小调整、旋转、片段剪辑的能力。

本文继续介绍我们的 MoviePy，将集中在 MoviePy 不同类型的 Clip 使用。

## 前言

MoviePy 中的 Clip 类，最基础的是 VideoClip 和 AudioClip，即视频与音频片段类。而在它之上，衍生出了几大类，分别是 VideoFileClip、AudioFileClip、TextClip、ImageClip 和 ColorClip。

## 

