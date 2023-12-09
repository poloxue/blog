---
title: "基于 Python 视频搬运 Part3 - 配置和数据的统一管理"
date: 2023-12-08T18:07:49+08:00
draft: false
comment: true
description: "本文介绍 mvideo 项目如何管理配置和视频搬运过程中的数据。"
---

本文介绍 mvideo 项目如何管理配置和视频搬运过程中的数据。

## 前言

在这个视频的处理过程中，我们会保存一些过程中的配置或者数据，如这是否是一个翻译类的项目，要处理 url 地址等等。

此外，处理过程中的数据，如视频的标题、描述等等，要需要保存下来，便于后续使用，而不是每次都要重复下载。

还有，为增强核心代码的可读性，提高代码的封装性和后续的扩展性，将如资源的下载目录，每个资源的路径统一管理，封装成一些特定的方法，将更易于使用。

## 资源目录

每个资源会下载当前目录下的以视频标题作为名称的子目录，其中的结构如下所示：

```bash
./cover.png
./how-to-learn-neovim-1/    # 视频标题小写同时空格替换为 -
  - video.webm              # 下载的视频文件
  - audio.mp4               # 下载的音频文件
  - audio.wav               # 从下载视频中抽取的 wav 文件，用于字幕识别
  - audio.srt               # 识别的字幕文件
  - audio_translate.srt     # 字幕的翻译文件
  - final.mp4               # 最终成品视频
```

如果每个功能函数都管理这些文件名，将着实很恼人，而且说不定还有写错。

## 管理类 Environment

我将定义一个类，名为 Environment，单独处理这类信息。

```python
class Environment:

  def __init__(self):
    self._project_path = os.path.abspath(".")

    self._config_path = os.path.join(self._project_path, "config.toml")
    self._config = (
        toml.load(open(self._config_path))
        if os.path.exists(self._config_path)
        else {}
    )
    self._urls = self._config.get("urls", [])
    self._translator = self._config.get("translator", {})

    self._data_path = os.path.join(self._project_path, "data.json")
    self._data = (
        json.load(open(self._data_path)) if os.path.exists(self._data_path) else {}
    )

    self._video_filename = "video.webm"
    self._audio_filename = "audio.mp4"
    self._audio_wav_filename = "audio.wav"
    self._audio_srt_filename = "audio.srt"
    self._trans_srt_filename = "audio_translate.srt"
    self._cover_image_filepath = "cover.png"
    self._final_filename = "final.mp4"

    self._videos = self._data.get("videos", {})

```

初始化阶段，首先会将必要的信息准备好，从配置文件 config.toml 读取配置，从 data.json 中读取数据。

还有，搬运过程中的一些乱七八糟的文件，如下载音视频文件名称，产生的字幕文件等等，也都定义在了这里。

## 辅助函数

Environment 还将一些这些属性通过辅助函数的形式提供出来。

如下是一些配置获取的辅助方法：

```python
    @property
    def project_path(self):
        return self._project_path

    def project_subpath(self, subpath):
        return os.path.join(self._project_path, subpath)

    def set_urls(self, urls):
        self._urls = urls

    def set_translator(self, translator, from_lang, to_lang):
        self._translator = {
            "translator": translator,
            "from_lang": from_lang,
            "to_lang": to_lang,
        }

    def translator(self, translator):
        if translator:
            return translator
        return self._translator.get("transaltor")

    def translator_from_lang(self, from_lang):
        if from_lang:
            return from_lang
        return self._translator.get("from_lang")

    def translator_to_lang(self, to_lang):
        if to_lang:
            return to_lang
        return self._translator.get("to_lang")

```

一些视频相关的辅助方法：

```python
    @property
    def videos(self):
        return self._videos

    def add_video(self, url, title, description):
        output_path = title.replace(" ", "-")
        video_path = os.path.join(self._project_path, output_path)
        self._videos[output_path] = {
            "url": url,
            "title": title,
            "path": video_path,
            "description": description,
            "video_filepath": os.path.join(video_path, "video.wepm"),
            "audio_filepath": os.path.join(video_path, "audio.mp4"),
            "audio_wav_filepath": os.path.join(video_path, "audio.wav"),
            "audio_srt_filepath": os.path.join(video_path, self._audio_srt_filename),
            "trans_srt_filepath": os.path.join(video_path, self._trans_srt_filename),
            "cover_filepath": os.path.join(video_path, self._cover_image_filepath),
            "final_filepath": os.path.join(video_path, self._final_filename),
        }

        return output_path

    @property
    def video_filename(self):
        return "video.webm"

    @property
    def audio_filename(self):
        return "audio.mp4"

    def video_filepath(self, output_path):
        return self._videos[output_path].get("video_filepath")

    def audio_filepath(self, output_path):
        return self._videos[output_path].get("audio_filepath")

    def audio_wav_filepath(self, output_path):
        return self._videos[output_path].get("video_filepath")

    def audio_srt_filepath(self, output_path):
        return self._videos[output_path].get("audio_srt_filepath")

    def trans_srt_filepath(self, output_path):
        return self._videos[output_path].get("trans_srt_filepath")

    def cover_filepath(self, output_path):
        return self._videos[output_path].get("cover_image_filepath")

    def final_filepath(self, output_path):
        return self._videos[output_path].get("final_filepath")
```

本质上，其实就是提供了一些 get 和 set 方法，写核心的代码更舒适一些。

如想获取成品视频文件路径，通过如下代码即可实现：

```python
env = Environment()
final_filepath = env.final_filepath("how-to-learn-neovim-part1/")
print(final_filepath)
```

## 更新写入

Environment 提供了很多 get set 方法，在这个视频处理过程中，可能会改变配置和数据文件，这时候，需要执行 env.flush 才能将内容写入到 config.toml 和 data.json 文件中。

为了防止异常中断写入，通过类似 try except 的方式保证写入成功。

```python
env = Environment()
try:
  env.set_urls(urls)
  ...
except Exception:
  pass
finally:
  env.flush()
```

## 最后

本文介绍了一个辅助类，用于统一管理全局的一些配置和过程中产生的数据，同时通过辅助函数提高代码的封装，便于后续扩展。

