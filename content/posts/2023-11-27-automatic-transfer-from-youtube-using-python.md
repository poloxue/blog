---
title: "基于 Python 视频搬运 Part 1 - 先导篇"
date: 2023-11-27T14:15:02+08:00
draft: false
comment: true
description: "本文介绍如何基于 Python 实现的从 YouTube 自动化搬运视频到国内平台的命令行工具，计划命名为 mvideo，即 move video 的意思。"
---

{{< video bb_id="621856735" >}}

本文介绍如何基于 Python 实现的从 YouTube 自动化搬运视频到国内平台的命令行工具，计划命名为 mvideo，即 move video 的意思。

这个工具我已经有了一个版本，但我想把它作为一个案例，把它做成一个系统化的工具，便于后续扩展，故而，就借着视频平台以视频的形式一步一步实现这第一个版本。

## 前言

话说，我为什么会想开发这样一款视频搬运工具呢？

出国的几年，在 Youtube 发现不少免费的教程视频。或许是因为 Youtube 广告机制收入不菲，与程序员有关的免费教程和频道非常之多。我就想着搬运一些视频分享到国内的小伙伴。

搬运的话，手动或自动化搬运皆可。但为搬运更多视频，能自动化肯定是最好的，而且技术视频搬运这事情也不挣啥钱，纯纯的慈善事业，还是要更多地专注于其他事情。

我对这个工具的期望是能支持从下载、字幕识别、翻译、字幕制作、封面制作，甚至是多视频合成，或者大视频拆分，最终自动上传。

另外，由于我希望搬运的视频是我看过的，所以我没有做自动监听频道直接搬运的能力。后期可以考虑，对于一些优质频道，无脑搬运也不是不行。

我从网上搜罗了不少资料，花了一星期的时间，最终写出了这个小工具。除了大视频的拆分，其他基本都已经支持了。还有，自动发布当前只支持 bilibili。 

我在 B 站顺便还开通了一个频道 - [Youtube技术视频](https://space.bilibili.com/313974328?spm_id_from=333.1007.0.0)，用于我的日常视频搬运。

## 方案

资源下载，Youtube 资源的下载使用的 pytube 实现，一款轻量的用于下载 Youtube 资源的 Python 包。

字幕制作，这其中主要涉及两点内容，分别是语音的文字识别和翻译。

字幕识别，使用 openai 开发开源的语音识别系统 whisper，它支持多种模型，断句不错，而且精度比视频平台默认的文字转语音准确率看起来更好。

字幕翻译，使用的是 python 的翻译库 [translators](https://github.com/uliontse/translators) ，它实现市面上大部分翻译渠道的对接，如 baidu, qqFanyi, google, bing 等都是支持的。如果想要高品质的翻译，则是需要花钱的，我当前只集成了 deepl 和 qqFanyi 两个付费的翻译器。

视频合成，包括封面和其他一些图片制作，字幕、音视频的合成，使用的是 python 的 moviepy 库实现，它基于如 ffmpeg 和其他一些图片、视频处理库的一个易用使用 Python 音视频剪辑库。

自动发布，通过 selenium 实现，当前支持持 B 站，不过，很多视频平台都有提供开放平台 OpenAPI，可以通过接口管理视频，暂时还没去暂时，到最后也可以考虑下。

## 项目需求

接下来，说明下这个项目的目标。毕竟，做项目要有需求的不是。

为了使项目的整体结构清晰，我将这个搬运项目不同功能拆解到了不同的命令上。效果上，有点以类似项目管理的方式。

核心四个基本步骤，如下所示：

init， 实现项目初始化和音视频素材下载；transcribe，用于生成原始和翻译字幕；make，用于合成成品视频； publish，用于发布视频到特定视频平台。

### 创建项目

新建视频搬运项目

```bash
mkdir project_directory
cd project_directory
mvideo init --urls "https://youtube.com/watch?v=xxxx" 
```

亦或是

```bash
mvideo init --playlist "https://www.youtube.com/playlist?list=xxxx"  --playlist-start 0 --playlist-end 3
```

该命令的作用是项目初始化，通过指定 url 下载音视频素材文件到项目下的视频标题命名的目录中。

项目相关的资源文件，如下载、制作过程中和最终成品的文件都会存放在项目目录下。如想要纠正机翻中的错误字幕，直接编辑对应的文件即可。

参数说明：

- \--urls，指定视频地址列表，指定单个或多个视频地址，多视频会自动合成一个视频；
- \--playlist，指定视频播放列表地址，优先于 urls；
- \--playlist-start/--playlist-end，指定下载播放列表范围，从哪个开始下载到哪个结束；

对于搬运翻译类的项目，可通过如下命令指定：

```bash
mvideo init --translator 'bing' --translator-from-lang 'en' --translator-to-lang 'zh'
```

- \--translator，指定翻译器，默认为 none，表示无，通过 ls-translators 列出所有翻译器；
- \--translator-from-lang，用于指定翻译的原始语言，默认为 en；
- \--translator-to-lang，用于指定翻译的目标语言，默认为 zh；

### 字幕制作

通过如下命令生成视频原始字幕：

```bash
mvideo transcribe --whisper-mode base
```

- \--whipser-mode，whisper 模型名，默认 base，可选 tiny, base, small, medium, large。

注：关于 whisper 的更多支持，请查看它的项目地址 [openai-whisper](https://github.com/openai/whisper)。

字幕是否翻译依据的是项目创建时的配置。但如果项目创建时 \--translator 指定了翻译器，但不希望翻译字幕，可通过如下选项单独配置：

- \--translator，指定翻译器，默认值取决于项目配置；
- \--translator-from-lang，用于指定翻译的原始语言，默认为 en；
- \--translator-to-lang，用于指定翻译的目标语言，默认为 zh；

### 视频合成

视频合成命令如下所示，将会按项目的默认配置将音视频、字幕合成到成一个成品视频。

```
mvideo make
```

如果希望在视频中增加封面、开始声明和结尾提醒，执行如下命令：

```bash
mvideo make --cover-text "封面标题" --declaim-text "特别说明：本频道专注于分享优质视频。" --end-text "感谢敢看，如您喜欢本视频，欢迎点赞、收藏、评论与关注 " --without-chapter
```

- \--cover-text，封面标题，视频开头，封面图存放于 output-path 目录下，如未指定则不制作；
- \--declaim-text，声明文字，声明文本，用于生成视频声明，未指定则无声明；
- \--end-text，结尾文本，用于感谢观众或提醒点赞关注等，未指定则无结尾提醒；

多视频合成时，默认会使用视频标题作为章节标题，如要禁用，可通过如下命令实现：

```
mvideo make --without-chapter
```

- \--without-chapter，禁用默认的章节提示；

视频合成的默认行为是，无论当前是否存在成品视频，都会重新合成视频。如想只在无成品视频的情况下才合成，使用如下命令：

```bash
mvideo make --smart
```

- \--smart，存在成品视频，跳过不合成，否则合成视频；

字幕合成配置，默认按项目配置，如果原始字幕和翻译字幕都存在，将会全部合成到视频中。

通过如下命令指定配置：

```
mvideo make --subtitle 'none'
```

- \--subtitle 指定合成的字幕，none-无，origin-原语言，translate-翻译，all-所有；

### 视频发布

视频发布，执行如下命令，会打开浏览器自动执行发布操作。

```bash
mvideo publish --platform bilibili
```

- \--platform，指定视频的发布平台，默认值为 bilibili 平台，当前只支持 bilibili；

执行发布的详细信息，如下所示：

```bash
mvideo publish --platform bilibili --title "发布视频主题" --keywords "程序员,编程"  --source-url "https://youtube.com/watch?v=xxx""
```

- title，指定视频发布标题，默认使用第一个视频标题，启用翻译的话，使用翻译版本；
- source-url，指定搬运来源，默认值，urls 的第一个视频地址或 playlist 即 playlist 地址；
- keywords，指定视频标签，默认使用 youtube 视频的第一个关键字；

## 最后

本文介绍了 YouTube 视频搬运的整体方案，下期视频将集中于介绍资源的下载。

我的博文：[基于 Python 视频搬运工具开发 Part 1](https://www.poloxue.com/posts/2023-11-27-automatic-transfer-from-youtube-using-python/)
