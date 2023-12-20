---
title: "基于 Python 视频搬运 Part6 - OpenAI whisper 语音识别"
date: 2023-12-09T20:25:35+08:00
draft: true
comment: true
description: "本文介绍如何使用 OpenAI whisper 进行语音文字识别。"
---

本文介绍如何基于 OpenAI 的开源模型 whisper 实现语音的文字识别。Whisper 是 OpenAI 开源的一款强大的语音识别模型，识别准确率完全不亚于甚至高于付费软件。

## 使用场景

语音的文字识别应用场景很多，常见的如视频的字幕识别，日常会议的语音转录。

看到网上的一个场景，留学的小伙伴不能实时听懂课程内容，会将课堂语音录制下来，用语音识别转录成文字。这不可不说也是语音识别的应用案例之一。

再者，看到这里，不禁想到，是否能实时转录，课上课后兼而有之，是不是更好呢？

本文也将做个尝试，基于 Python 和 Whisper 实现实时转录效果。如果将它用于一些会议或课堂上，对于听力水平一般的朋友，不可谓是雪中送炭啊。

## 如何安装

Whipser 的安装非常简单。如果是 Mac 用户，且系统已经安装了 Python 环境，可通过如下两个命令安装。

```bash
pip install whisper
brew install ffempg
```

其他系统 whisper 的安装命令相同，ffempg 的安装方法，通过系统内置的包管理器即可安装。

命令如下所示：

```bash
# on Ubuntu or Debian
sudo apt update && sudo apt install ffmpeg

# on Arch Linux
sudo pacman -S ffmpeg

# on MacOS using Homebrew (https://brew.sh/)
brew install ffmpeg

# on Windows using Chocolatey (https://chocolatey.org/)
choco install ffmpeg

# on Windows using Scoop (https://scoop.sh/)
scoop install ffmpeg
```

到此，whisper 顺利安装成功。

## Whisper 模型

Whisper 是免费开源模型，它是基于从网络上收集的 680,000 小时多语言数据进行的训练。它提供了 5 种模型规模，分别是 tiny、base、small、medium 和 large。

如下是模型的参数、大小和内存要求等信息的一览表。

Size	 | Parameters	| English-only model	| Multilingual model	| Required VRAM	| Relative speed
------ | ----------- | ------------------- | ------------------- | ------------- | ----------------
base	 | 74 M	      | base.en	            | base	              | ~1 GB         | ~16x 
tiny	 | 39 M	      | tiny.en	            | tiny	              | ~1 GB	        | ~32x
small	 | 244 M	    | small.en	          | small	              | ~2 GB	        | ~6x
medium | 769 M	    | medium.en	          | medium	            | ~5 GB	        | ~2x
large	 | 1550 M	    | N/A	                | large	              | ~10 GB        | 1x

whisper 的默认模型是 small，如果语音的质量较高，不同模型的识别差异并不大，但如果语音质量较低，如有噪音、发音不清晰等，亦或希望高质量的断句，则需要切换为更大的模型。

一般情况下，medium 模型已经使用了。

如果想用 large 模型，由于它对硬件要求较高，且速度较慢，最好有较好的硬件支持。亦或者也可直接使用 OpenAI 开放 API 提供的语音文字能力。具体如何使用，本文后面会有介绍。

## 如何使用

whisper 支持两种使用方法，即分别是基于命令和基于代码库。

### 命令行

首先，基于命令行的使用。假设，我的当前目录有一个语音文件名为 audio.wav，如何转录？

命令如下所示：

```bash
whisper audio.wav
```

识别过程中，会同时实时输出结果，如下所示：

```bash
Detecting language using up to the first 30 seconds. Use `--language` to specify the language
Detected language: English
[00:00.000 --> 00:05.000]  Hey everyone, today we'll be talking, I'm sorry, is it fine if I record a video?
[00:05.000 --> 00:06.000]  No, thank you so much.
[00:18.000 --> 00:22.000]  Usually when you're building a system, an engineering system, there's actually some sort of background behind it.
[00:22.000 --> 00:26.000]  We'll be taking a real world example of opening a restaurant. Let's see how that happens.
...
```

非常 easy 是不是。

命令执行完成后，会在当前目录生成结果文件，如下所示：

```bash
$ ls
audio.json  audio.srt  audio.tsv  audio.txt  audio.vtt  audio.wav
```

除了 audio.wav，其他都是生成的结果文件，只是文件格式不同而已。

### Whisper 库

从前面的安装步骤推测，whipser 提供了 python 库供我们调用。

## 更多配置

本小节介绍 whisper 的配置项。whipser 提供了众多选项，在命令行和代码中，我们均可设置。

想了解它的所有选项，可通过 `--help` 选项自行查看。

```bash
whisper --help
usage: whisper [-h] [--model MODEL] [--model_dir MODEL_DIR] [--device DEVICE]
               [--output_dir OUTPUT_DIR] [--output_format {txt,vtt,srt,tsv,json,all}]
               [--verbose VERBOSE] [--task {transcribe,translate}]
               [--language {af,am,ar,as,az,ba,be,bg,bn,bo,br,...}]
                ...
               [--highlight_words HIGHLIGHT_WORDS] [--max_line_width MAX_LINE_WIDTH]
               [--max_line_count MAX_LINE_COUNT]
               [--max_words_per_line MAX_WORDS_PER_LINE] [--threads THREADS]
               audio [audio ...]

```

下面开始介绍它的一些常用的配置项。

### 语言配置

我们前面的例子中由于未指定语言，会看到如下这行内容。

```bash
Detecting language using up to the first 30 seconds. Use `--language` to specify the language
Detected language: English
```

whisper 默认为自动检测开头 30 秒内容作为转录的目标语言。如果想省略这一步可通过 `--language` 选项指定语言。

命令如下所示：

```bash
whisper audio.wav --language English
```

whisper 支持的语言中断，在其源码文件中 [tokenizer.py](https://github.com/openai/whisper/blob/main/whisper/tokenizer.py) 中可查看所支持的语言列表。

### 任务选项

前面提到 whipser 支持两种类型的任务，转录 transcribe 和翻译 translate，默认行为是 transcribe。要改变这个默认行为，可通过选项 `--task` 改变。

命令如下所示：

```bash
whisper audio.wav --task translate
```

### 模型类别

whisper 的默认模型是 small，如果想要更好的效果，减少如噪音影响或提高断句水平等，可通过选项 `--model` 切换模型。

如下将识别模型切换为 medium：

```bash
whisper audio.wav --model medium
```

本地使用的话，medium 一般已经足够使用了。

## 输出格式

当一个 whisper 任务运行结束，默认会在当前目录下生成如 audio.srt、audio.txt、audio.tsv、audio.vtt 等不同格式的文件。

如下所示：

```bash
audio.json  audio.srt  audio.tsv  audio.txt  audio.vtt  audio.wav
```

熟悉字幕格式的话，就知道 srt，这是很多视频平台都支持的字幕格式，这里的 audio.srt 可直接作为我们的字幕文件。就这一点，将极大提效我们视频制作的生产力。

whisper 默认会将所支持的格式文件都输出一份，如果我们只要某一个格式，可通过选项 `--output_format` 指定。

例如，一般情况下，我只想要 srt 格式文件，命令如下：

```bash
whisper audio.wav --output_format  srt
```

### 字幕长度

常用的 whisper 选项介绍已经差不多了。我再补充一个重要的点，即如何限制输出文本的每行长度。基本没有文章介绍这一点，我也是因为实际使用时，遇到了这个问题。

对字幕而言，如果不限制文本长度，可能会出现太长的文本，有碍观瞻。我们可通过组合使用 whisper 几个选项，实现这个能力。

首先，word timestamp，即单词级别的时间戳。

