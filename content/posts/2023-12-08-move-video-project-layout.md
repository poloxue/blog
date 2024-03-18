---
title: "基于 Python 视频搬运 Part2 - 代码布局"
date: 2023-12-02T01:18:23+08:00
draft: false
comment: true
description: "本文介绍基于 Python 视频搬运项目的代码布局。"
---

本文介绍基于 Python 视频搬运项目的代码布局。

## 前言概述

项目的代码布局要从需求出发，一方面是既要满足当前的项目功能，也能保证一定的结构性便于后续扩展代码。

这个工具本质是一个命令行工具，我在 [先导篇](http://localhost:1313/posts/2023-11-27-automatic-transfer-from-youtube-using-python/) 中介绍了该项目的目标。

我们用了大量的子命令，我将用 click 这个 python 包解耦分离这个命令的功能。关于 click 的介绍，可查看其 [官方文档](https://click.palletsprojects.com/en/8.1.x/)。

## 命令布局

本项目核心子命令一共 4 个，分别是 init、trascribe、make 和 publish，统一使用 click 包的 click.group 包裹为子命令。

与之相应的一些核心文件的分布情况，如下所示：


```bash
- mvideo/__init__.py
- mvideo/main.py
- mvideo/cmds/__init__.py
- mvideo/cmds/init.py
- mvideo/cmds/transcribe.py
- mvideo/cmds/make.py
- mvideo/cmds/publish.py
```

cmds 目录下是所有我们要实现的子命令。

### main 文件

main.py 中是命令的入口文件，用于定义 main 命令。

代码如下所示：

```python
import click

import mvideo.cmds as cmds

click.group()
def cli():
  pass


def main():
  cli.add_command(cmds.transcribe)
  cli.add_command(cmds.init)
  cli.add_command(cmds.make)
  cli.add_command(cmds.publish)
  cli()

if __name__ == "__main__"
  main()
```

我们将 init, transcribe, make 和 publish 挂在了 cli 命令组下面。

### init 初始化下载资源

init 初始化下载资源，代码如下：

```python
import click


@click.command("init")
@click.option("--urls", type=click.STRING, help="The list of video URL")
@click.option("--playlist", type=click.STRING, help="Playlist URL")
@click.option("--playlist-start", type=click.INT, help="Playlist start index")
@click.option("--playlist-end", type=click.INT, help="Playlist end index")
@click.option("--translator", type=click.STRING, help="Translator")
@click.option("--translator-from-lang", type=click.STRING, help="Translator from lang")
@click.option("--translator-to-lang", type=click.STRING, help="Translator to lang")
def init(
    urls,
    playlist,
    playlist_start,
    playlist_end,
    translator,
    translator_from_lang,
    translator_to_lang,
):
    pass
```


init 参数说明：

- \--urls，指定视频地址列表，指定单个或多个视频地址，多视频会自动合成一个视频；
- \--playlist，指定视频播放列表地址，优先于 urls；
- \--playlist-start/--playlist-end，指定下载播放列表范围，从哪个开始下载到哪个结束；
- \--translator，指定翻译器，默认为 none，表示无，通过 ls-translators 列出所有翻译器；
- \--translator-from-lang，用于指定翻译的原始语言，默认为 en；
- \--translator-to-lang，用于指定翻译的目标语言，默认为 zh；

### transcribe 音频转录与翻译

transcribe 代码如下所示：

```python
@click.command("transcribe")
@click.option("--whisper-mode", type=click.STRING, help="whisper model")
@click.option("--translator", type=click.STRING, help="Translator")
@click.option("--translator-from-lang", type=click.STRING, help="Translator from lang")
@click.option("--translator-to-lang", type=click.STRING, help="Translator to lang")
def transcribe(whisper_mode, translator, translator_from_lang, translator_to_lang):
    pass
```

transcribe 参数说明：

- \--whipser-mode，whisper 模型名，默认 base，可选 tiny, base, small, medium, large。
- \--translator，指定翻译器，默认值取决于项目配置；
- \--translator-from-lang，用于指定翻译的原始语言，默认为 en；
- \--translator-to-lang，用于指定翻译的目标语言，默认为 zh；

### make 合成制作视频

make 制作视频，代码如下：

```python
@click.command("make")
@click.option("--cover-text", type=click.STRING, help="Covert text")
@click.option("--declaim-text", type=click.STRING, help="Declaim text")
@click.option("--end-text", type=click.STRING, help="End text")
@click.option("--without-chapter", is_flag=True, help="Without chapter")
@click.option("--smart", is_flag=True, help="If final video exists, dont override")
@click.option(
    "--subtitle-mode",
    type=click.STRING,
    help="Subtitle, options: all, origin, translate, none",
)
def make(cover_text, declaim_text, end_text, without_chapter, smart, subtitle_mode):
    pass
```

make 参数说明；

- \--cover-text，封面标题，视频开头，封面图存放于 output-path 目录下，如未指定则不制作；
- \--declaim-text，声明文字，声明文本，用于生成视频声明，未指定则无声明；
- \--end-text，结尾文本，用于感谢观众或提醒点赞关注等，未指定则无结尾提醒；
- \--without-chapter，禁用默认的章节提示；
- \--smart，存在成品视频，跳过不合成，否则合成视频；

### publish 发布视频

```python
import click


@click.command("publish")
@click.option(
    "--platform", type=click.STRING, help="Platform where you want to publish"
)
@click.option("--title", type=click.STRING, help="Title of video")
@click.option("--source-url", type=click.STRING, help="Origin url of this video")
@click.option("--keywords", type=click.STRING, help="Keywords of this video")
def publish(platform, title, source_url, keywords):
    pass
```

- \--platform，指定视频的发布平台，默认值为 bilibili 平台，当前只支持 bilibili；
- \--title，指定视频发布标题，默认使用第一个视频标题，启用翻译的话，使用翻译版本；
- \--source-url，指定搬运来源，默认值，urls 的第一个视频地址或 playlist 即 playlist 地址；
- \--keywords，指定视频标签，默认使用 youtube 视频的第一个关键字；

## 最后

本文介绍了 mvideo 视频搬运项目的代码布局结构，以及主要子命令的入口代码，接下来的重点就是结合如何实现每个子命令的功能。
