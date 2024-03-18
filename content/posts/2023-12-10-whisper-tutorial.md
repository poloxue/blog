---
title: "语音识别 OpenAI whisper 完全指南"
date: 2023-12-09T20:25:35+08:00
draft: true
comment: true
description: "本文介绍如何使用 OpenAI whisper 进行语音文字识别。"
---

本文介绍一款免费强大的语音识别模型 - whisper，由 OpenAI 开源。它的识别准确率完全不亚于甚至高于付费软件。

## 前言

最近用 whisper 给英文视频加字幕，整体的单词识别正确率还是很高的。除了会出现断句不准确，但这会导致翻译不准确，以至提高校验字幕的时间成本。

本文将从一个用户角度，介绍如何使用 whisper，包括一些代码使用细节，和我使用过程中所遇到的一些问题等，从而提高 whisper 的使用效率。

## 使用场景

语音识别的应用场景很多，常见的如视频的字幕识别，日常会议的语音转录。

在网上看到一个场景，留学的小伙伴由于不能实时听懂课堂内容，会将课堂语音录制下来，语音识别转录成文字。这不可不说也是语音识别的应用案例之一。

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

如下是模型的大小，参数量和内存要求等信息的一览表。

Size	 | Parameters	| English-only model	| Multilingual model	| Required VRAM	| Relative speed
------ | ----------- | ------------------- | ------------------- | ------------- | ----------------
base	 | 74 M	      | base.en	            | base	              | ~1 GB         | ~16x 
tiny	 | 39 M	      | tiny.en	            | tiny	              | ~1 GB	        | ~32x
small	 | 244 M	    | small.en	          | small	              | ~2 GB	        | ~6x
medium | 769 M	    | medium.en	          | medium	            | ~5 GB	        | ~2x
large	 | 1550 M	    | N/A	                | large	              | ~10 GB        | 1x

whisper 的默认模型是 small，如果语音的质量较高，不同模型的识别差异并不大，但如果语音质量较低，如有噪音、发音不清晰等，亦或希望高质量的断句，则需要切换为 medium 或 large 模型。

一般情况下，medium 模型已经使用了。large 模型对硬件要求较高，速度慢。当然，也可使用 OpenAI 开放 API，有提供的语音识别接口，使用的是 large 模型。具体使用方法，本文后面会介绍。

## 如何使用

whisper 提供了两种使用方法，分别是基于命令行和基于它的代码库。

### 命令行

首先，基于命令行的使用。

假设，我的当前目录有一个语音文件名为 audio.wav，如何转录？

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

输出内容的基本格式：start 开始时间 --> end 结束时间 text 语音识别的文本内容。

命令执行完成后，会在当前目录生成结果文件，如下所示：

```bash
$ ls
audio.json  audio.srt  audio.tsv  audio.txt  audio.vtt  audio.wav
```

除了 audio.wav，其他都是生成的结果文件，只是文件格式不同而已。其中的 srt 就是字幕文件常用的格式。

### 代码库

从前面的安装步骤可知，whipser 其实提供了 python 代码库，我们可以直接通过调用它的函数实现语言识别。

接下来，具体演示下它的使用。演示案例代码，如下所示：

```python
import whisper

model = whisper.load_model('medium')
result = model.transcribe("audio.wav")
print(result)
```

我们导入的是 medium 模型，使用 transcribe 函数转录语音为文本。

输出结果，如下所示：

```bash
{
  'text': " Array Reduce, start with a list of items, then iterate over them to compute a single value.  It works just like basic arithmetic. Consider 1 plus 2 plus 3. What we have here is a list of numbers. ...",
  'segments': [
    {
      'id': 0, 'seek': 0, 'start': 0.0, 'end': 5.34,
      'text': ' Array Reduce, start with a list of items, then iterate over them to compute a single value.',
      'tokens': [50364, 1587, 3458, 4477, 4176, 11, 722, 365, 257, 1329, 295, 4754, 11, 550, 44497, 670, 552, 281, 14722, 257, 2167, 2158, 13, 50631],
      'temperature': 0.0,
      'avg_logprob': -0.15513807443472055,
      'compression_ratio': 1.6609589041095891,
      'no_speech_prob': 0.05010032653808594
    }, 
    {...},
    {...}
  ],
  'language': 'en'
}
```

其中 text 是完整的文本内容，而 segments 是文本分片，包含 start 开始时间、end 结束时间和 text 文本内容。

其他几个参数，一般我也不看，简单说明如下：

- temperature: 控制文本输出多样性和随机性的参数；
- avg_logprob: 模型预测的置信度评分的平均值；
- compression_ratio: 音频信号压测的比率；
- no_speech_prob: 模型检测没有语音的概率；

如果是翻译任务，使用 translate 函数，代码如下：

```python
result = model.translate("audio.wav")
```

whisper 还可以利用开始的 30 秒的语音识别出它的语言。

具体代码如下所示：

```python
import whipser

model = whisper.load_model("base")

# load audio and pad/trim it to fit 30 seconds
audio = whisper.load_audio("audio.mp3")
audio = whisper.pad_or_trim(audio)

# make log-Mel spectrogram and move to the same device as the model
mel = whisper.log_mel_spectrogram(audio).to(model.device)

# detect the spoken language
_, probs = model.detect_language(mel)
print(f"Detected language: {max(probs, key=probs.get)}")
```

输出结果，如下所示：

```bash
Detected language: en
```

传递配置选项也很方便，直接传递参数给 transcribe 函数即可。

示例代码如下：

```python
model.transcribe('audio.wav', word_timestamps=True, language='en')
```

配置了 word_timestamps 和 language 两个选项。

## 配置选项

本小节介绍 whisper 的配置项。whipser 提供了众多配置选项，在命令行和代码中，我们均可设置。

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

如上的命令行配置选项，代码层面，大部分都可以按前面介绍的，通过 transcribe 或 translate 任务函数传递配置项即可。

下面开始介绍它的一些常用的配置项。

### 语言配置

我们前面的例子中由于未指定语言，会看到如下这行内容。

```bash
Detecting language using up to the first 30 seconds. Use `--language` to specify the language
Detected language: English
```

whisper 默认为自动检测开头 30 秒内容作为转录的目标语言。如果想跳过这一步可通过 `--language` 选项指定语言。

命令如下所示：

```bash
whisper audio.wav --language en
```

代码如下所示：

```python
model.transcribe(language='en')
```

whisper 支持的语言种类众多，可在源码文件 [tokenizer.py](https://github.com/openai/whisper/blob/main/whisper/tokenizer.py) 中可查看支持的语言列表。

### 任务选项

前面提到 whipser 支持两种类型的任务，转录 transcribe 和翻译 translate，默认行为是 transcribe。要改变这个默认行为，可通过选项 `--task` 改变。

命令如下所示：

```bash
whisper audio.wav --task translate
```

代码的实现，如下所示：

```python
model.transcribe('audio.wav')
model.translate('audio.wav')
```

通过函数直接切换任务类型。

### 模型类别

whisper 的默认模型是 small，如果想要更好的效果，减少如噪音影响或提高断句水平等，可通过选项 `--model` 切换模型。

如下将识别模型切换为 medium：

```bash
whisper audio.wav --model medium
```

代码实现，如下所示：

```python
model = whipser.load('meidum')
mode.transcribe('audio.wav')
```

### 初始提示

whisper 支持提供初始提示 initial-prompt 优化模型，提高识别效果。

initial prompt 的常见使用场景，有如模型中有一些专业术语，即可通过提示告诉模型，还有如断句，如希望将转录的句子翻译，最后是一句完整的话，也可通过提示告诉 whipser。

如没有带上提示的效果如下：

```plain
[00:09.360 --> 00:14.080]  What we have here is a list of numbers. They're reduced to a single value using a function called
[00:14.080 --> 00:18.800]  Addition. In arithmetic, we go from left to right. The result of the first computation
[00:18.800 --> 00:23.680]  is then added to the next one, so it's actually just a loop, but a loop with a memory.
[00:23.680 --> 00:28.240]  Take, for example, an array of orders from our online store. Our goal is to calculate the total
```

带上提示的效果入校：

```bash
$ whisper audio.wav --initial_prompt "Hi, guys. This tutorial will explain Array Reduce in 100 seconds."
...
[00:09.000 --> 00:15.000]  What we have here is a list of numbers. They're reduced to a single value using a function called Addition.
[00:15.000 --> 00:20.000]  In arithmetic, we go from left to right. The result of the first computation is then added to the next one.
[00:20.000 --> 00:23.000]  So it's actually just a loop, but a loop with a memory.
[00:23.000 --> 00:26.000]  Take, for example, an array of orders from our online store.
[00:26.000 --> 00:31.000]  Our goal is to calculate the total dollar amount of sales across all of the orders in this array.
...
```

明显发现，加上 initial prompt 后，识别的断句效果提升了很多。这对于后面翻译的效果也会有很大的提升。

### 输出格式

当一个 whisper 任务运行结束，默认会在当前目录下生成如 audio.srt、audio.txt、audio.tsv、audio.vtt 等不同格式的文件。

如下所示：

```bash
audio.json  audio.srt  audio.tsv  audio.txt  audio.vtt  audio.wav
```

熟悉字幕格式的话，就知道 srt 是很多视频平台都支持的字幕格式。

这里的 audio.srt 可直接作为我们的字幕文件，导入视频平台或一些视频剪辑软件即可。就这一点，就能极大提效我们视频制作效率。

whisper 默认会将所支持的格式文件都输出一份，如果我们只要某一个格式，可通过选项 `--output_format` 指定。

例如，一般情况下，我只想要 srt 格式文件，命令如下：

```bash
whisper audio.wav --output_format  srt
```

代码实现部分，输出特定格式的文件这部分的代码实现要稍微看下它的命令实现。

whisper 代码库的中输出文件有不同的 Writer 实现，如 SRT 文件的写入实现类是 WriteSRT，JSON 有  WriteJSON 等。

whipser 中提供了一个函数 get_writer，按 output_format 返回相应格式的 Writer。

实现写入 SRT 文件，代码示例如下所示：

```python
from whisper.utils import get_writer 

audio_path = 'audio.wav'

model = model.load(audio_path)
result = model.transcribe(audio_path)

writer = get_writer(output_format='srt', output_dir='.')
writer(result, audio_path)
```

### 字幕长度

常用的 whisper 选项介绍已经差不多了。我再补充一个重要的点，即如何限制输出文本的每行长度。

如果字幕不限制文本长度，可能会出现太长的情况，降低视频的观看体验。我们可通过组合 whisper 的几个选项，实现这个能力。

- word_timestamp，即单词级别的时间戳，启用了这个选项才能在限制文本长度的同时，保证字幕的时间戳是对的。
- max_words_per_line，限制每行最大的单词数量，依赖于 word_timestamps 为 True；
- max_line_width，限制每行字符长度，依赖于 word_timestamps 为 True；
- max_line_count，限制每个句子字幕显示的最大行数，与 max_line_width 配合使用，依赖于 word_timestamps 为 True；

演示几个案例吧。

命令行示例，限制每行最多 10 个单词或限制字幕两行展示，每行长度不超过 50；

```bash
whisper audio.wav --word_timestamps True --max_words_per_line 10
whisper audio.wav --word_timestamps True --max_line_width 50 --max_line_count 2
```

代码示例

```python
result = model.transcribe('audio.wav', word_timestamps=True)
writer(result, audio_path, max_words_per_line=10)
writer(result, audio_path, max_line_width=50, max_line_count=2)
```

这个配置项只对输出文件有效果，查看如 audio.srt 产看效果。

限制字幕的单词水量的的效果如下所示：

限制字幕的最大行数和宽度的效果如下所示：

```bash
1
00:00:00,000 --> 00:00:05,360
Array Reduce. Start with a list of items, then
iterate over them to compute a single value. It

2
00:00:05,360 --> 00:00:11,100
works just like basic arithmetic. Consider 1, plus
2, plus 3. What we have here is a list of numbers.
```

但如果希望结合翻译，这种方式就不是很方便，与翻译字幕的联动效果实现起来会麻烦一些。

## OpenAI 接口

Whisper 是免费的开源模型，但如果受限于硬件条件，想要更快的速度，可使用 OpenAI 的语音识别开放接口。

通过一个案例介绍它的使用，代码如下所示：

```python
from openai import OpenAI

client = OpenAI()
result = client.audio.transcriptions.create(file=open("audio.wav"), response_format='srt')
```

> 注：使用 OpenAI 的 API 的前提，你得有个 OpenAI 的账号，且已经设置好了 OPNEAI_API_KEY 的环境变量。

result 即使我们要的文本，直接写入文件即可。

在使用 OpenAI 的接口时，遇到了一些的问题。

首先是超时问题，如果网速不够，稍微大一些的文件，容易超时，通过 httpx.Timeout 可将超时时间调为更大的数值。

示例代码如下所示：

```python
import httpx

result = client.audio.transcriptions.create(
  file=open("audio.wav"),
  response_format='srt',
  timeout=httpx.Timeout(600, read=300, write=300, connect=3, pool=300),
)
```

但即使这个问题解决了，还有一个问题，即上传文件有大小限制，最大 25 M。

第一种解决方案，使用压缩率高的编码格式，如 mp3，我们可以通过 pydub 将默认的 wav 转为 mp3 格式，降低文件大小。

示例代码如下：

```python
import io
from pydub import AudioSegment

## other import statements 其他导入语句

audio_buffer = io.BytesIO()
audio_buffer.name = "audio.mp3"
AudioSegment.from_wav("audio.wav").export(audio_buffer, format="mp3")

result = client.audio.transcriptions.create(
  file=audio_buffer,
  response_format='srt',
  timeout=httpx.Timeout(600, read=300, write=300, connect=3, pool=300),
)
```

这里使用 io 包中的 BytesIO 缓存转化为的 mp3 格式数据。

但其实问题还没有根本解决，因为 mp3 格式压缩大小，但大文件依然还是会超过大小限制。

永久解决方式是，通过 pydub 切分语音文件，如切分为十分钟一个文件，再上传。

示例代码，如下所示：

```python
from pydub import AudioSegment
```

## 实时转录
