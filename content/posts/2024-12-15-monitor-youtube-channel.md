---
title: "效率神器：用 Python 监控油管频道，AI 生成文章"
date: 2024-12-15T17:17:48+08:00
draft: false
comment: true
description: "油管是很多人获取专业知识和行业资讯的渠道，有很多有价值的信息。对于要深入学习的内容，我们肯定要看完整个视频，而如果你只是关注一些资讯时间或分析视频，跟进热点信息，比如我平时关注一些财经币圈相关的内容资讯，花时间看完每个视频，效率地下。还有，油管上有不少高质量的英文视频，看起来有点吃力，将其转为中文，理解起来更容易。"
---

油管是很多人获取专业知识和行业资讯的渠道，有很多有价值的信息。对于要深入学习的内容，我们肯定要看完整个视频，而如果你只是关注一些资讯时间或分析视频，跟进热点信息，比如我平时关注一些财经币圈相关的内容资讯，花时间看完每个视频，效率地下。还有，油管上有不少高质量的英文视频，看起来有点吃力，将其转为中文，理解起来更容易。

现在市面上有没有这样的工具呢？

我知道有些 AI 工具，进到视频播放页面，用这些 AI 工具可以分析总结，但我很赖，压根不想去到某个频道查看它是否有新视频更新。我没有找到这类监听某个频道的分析推送的，或许即使有，费用应该也不低吧。


这篇文章我将实现这样一个小工具，它能监控我指定的某个频道，还能把监控到最新视频内容提取成文章，用邮件推送给我们。我将用 Python 完成这一系列工作。

---

## **目标是什么？**

先把目标明确下来，这个工具要实现以下功能：

1. **频道检测**：检查感兴趣油管频道是否有新的视频发布；
2. **下载音频**：检测到新视频后，只下载它的音频部分，节省存储空间；
3. **提取字幕**：提取音频内容中的文字；
4. **生成文章**：将提取的字幕整理为通俗易懂的文章；

这样，我就不需要逐个观看每个视频，而是直接阅读整理后的文章。

---

## **用到哪些工具？**

为了实现上述的目标，我们还需要几个重要的 Python 库：

1. **yt-dlp**：一个强大的 YouTube 下载工具，可以获取视频元数据或音频；
2. **Whisper**：OpenAI 的音频转文字模型，用于从音频中提取字幕文字；
3. **OpenAI**：用来调用 OpenAI 的接口，将识别出来的字幕转换为通俗易懂的文章；

如下命令安装依赖包：

```bash
pip install yt-dlp openai-whisper openai
```

---

开始具体的实现吧！我将按步骤进行。

## **监控油管频道**

油管频道页面会列出了最新发布的视频，通过 `yt-dlp` 工具来抓取这些信息。

代码示例：

```python
import click
import yt_dlp

def fetch_latest_videos(channel_url, fetch_count=5):
  ydl_opts = {
    "quiet": True,
    "extract_flat": "in_playlist",
    "playlist_items": f"1-{fetch_count}",
  }
  with yt_dlp.YoutubeDL(ydl_opts) as ydl:
    info = ydl.extract_info(channel_url, download=False)
    return [{"title": entry["title"], "url": entry["url"]} for entry in info["entries"]]

@click.command()
@click.option("--channel-url", type=click.STRING, help="Channel URL")
@click.option("--count", default=5, type=click.INT, help="Video Count")
def main(channel_url, count):
  videos = fetch_latest_videos(channel_url, count)
  for video in videos:
    print(f'title:{video["title"]}, url: {video["url"]}')


if __name__ == "__main__":
  main()
```

执行如下命令监听某个频道：

```bash
python index.py --channel-url https://www.youtube.com/@CoinBureau/videos
```

输出最新的 5 个视频：

```bash
title:Wall Street Elites Are Going To Pump Crypto: Find Out Who!!, url: https://www.youtube.com/watch?v=WQCCwC2-Wkw
title:Is Trump Media The Next MicroStrategy!? TruthFi, Bakkt, & More!!, url: https://www.youtube.com/watch?v=c2-Gf473kmQ
title:APT to 10x?! Aptos Updates & Predictions You CANT Miss!!, url: https://www.youtube.com/watch?v=n39P_F6Cx58
title:Why Europe Is Falling Apart—and What It Means for YOU, url: https://www.youtube.com/watch?v=qB2QaYXXoBo
title:Crypto’s Dark Secret Exposed: Why 75% of Investors FAIL!, url: https://www.youtube.com/watch?v=-ZNXm3nX3zM
```

上面这段代码会提取频道最新的视频列表，包括标题和链接。我们只提取元信息，并不下载视频内容。其中的 `fetch_count` 是用来指定获取最新视频的数量。

---

## **下载音频文件**

有了视频链接后，我们用 `yt-dlp` 只下载音频文件，这样节省空间，简单高效。

示例代码：

```python
def download_audio(video_url, output_dir):
  ydl_opts = {
    "format": "bestaudio/best",
    "postprocessors": [
      {
        "key": "FFmpegExtractAudio",
        "preferredcodec": "mp3",
        "preferredquality": "192",
      }
    ],
    "outtmpl": f"{output_dir}/%(id)s/audio.%(ext)s",
    "download_archive": "downloaded_videos.txt", # 防止重新下载
  }
  with yt_dlp.YoutubeDL(ydl_opts) as ydl:
    info_dict = ydl.extract_info(video_url, download=True)
    if info_dict is not None and "requested_downloads" in info_dict:
      return info_dict["title"], info_dict["requested_downloads"][0]["filepath"]
      else:
      return f"{output_dir}/{video_url.split('v=')[-1]}/audio.mp3"

```

这个函数的功能包括：

- 实现音频部分的下载，将下载文件统一转换为 MP3 格式；
- 输出文件会保存在指定目录中，目录名为为视频 ID；
- 下载函数的返回值为下载音频的文件路径；

将这部分功能合并到主函数中。

```python
@click.command()
@click.option("--channel-url", type=click.STRING, help="Channel URL")
@click.option("--count", default=5, type=click.INT, help="Video Count")
@click.option("--output-dir", default=".", type=CLICK.STRING, help="Output Directory")
def main(channel_url, count, output_dir):
  videos = fetch_latest_videos(channel_url, count)
  for video in videos:
    download_audio(video["url"], output_dir=output_dir)
```

再次执行脚本就可以下载你指定频道的音频文件了。

```bash
$ python index.py --channel-url https://www.youtube.com/@CoinBureau/videos
```

---

## **提取字幕**

下载了音频文件后，我们通过 OpenAI 的 Whisper 模型，就可以从音频中提取出文本字幕，whisper 的使用很简单。

代码示例：

```python
import os
import whisper

def transcribe_audio(audio_path):
  subtitle_path = audio_path.replace("audio.mp3", "subtitles.txt")
  if os.path.exists(subtitle_path):
    with open(subtitle_path) as fd:
      return fd.read()

    model = whisper.load_model("base")
    result = model.transcribe(audio_path)
    subtitles = result["text"]

    with open(subtitle_path, "w+") as fd:
      fd.write(subtitles)
    return subtitles
```

我们使用 Whisper 的 base 模型识别音频内容，它的返回 `result[text]` 就是我们要的完整的字幕文本。上面的代码为了防止重复识别，加了去重判断逻辑，如果已识别字幕，直接访问文字即可。

将这部分合并到主函数流程中，如下：

```python
def main(channel_url, count, output_dir):
    videos = fetch_latest_videos(channel_url, count)
    for video in videos:
        audio_path = download_audio(video["url"], output_dir=output_dir)
        subtitles = transcribe_audio(audio_path)
        print(subtitles)
```

如果你是首次使用 whisper 识别音频文字，它会下载模型文件，大小视你使用的模型。

---

## **生成文章**

有了字幕，现在就可以借助 GPT 模型将器整理成逻辑清晰、易于阅读的文章。

代码示例：

```python
import openai

ai_client = openai.OpenAI()


def generate_article(audio_path, subtitles, title, video_url):
    article_path = audio_path.replace("audio.mp3", "article.md")
    if os.path.exists(article_path):
        with open(article_path) as fd:
            return fd.read()

    response = ai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {
                "role": "user",
                "content": f"以下是视频的字幕内容：\n\n{subtitles}\n\n请根据这些字幕生成一篇文章，原始主题是：{title}",
            },
        ],
        temperature=0.7,
    )
    article = response.choices[0].message.content
    article = f"来源：[{title}]({video_url})\n\n{article}"
    with open(article_path, "w+") as fd:
        fd.write(article)
    return article
```

将字幕内容和提示词交易 gpt 模型，最终就能得到我们心心念念的最终结果。

将这段逻辑加到主函数，如下：

```python
@click.command()
@click.option("--channel-url", type=click.STRING, help="Channel URL")
@click.option("--count", default=5, type=click.INT, help="Video Count")
@click.option("--output-dir", default=".", type=click.STRING, help="Output Directory")
def main(channel_url, count, output_dir):
    videos = fetch_latest_videos(channel_url, count)
    for video in videos:
        audio_path = download_audio(video["url"], output_dir=output_dir)
        subtitles = transcribe_audio(audio_path)
        article = generate_article(audio_path, subtitles, video["title"], video["url"])
        print(article)
```

最核心的部分全部完成了。

## 注意点

- openai 是要 APIKEY 的，我已经设置了环境变量，关于 openai 权限，需要各位自己想办法。
- 如果字幕太大，openai 没有办法直接处理，可以考虑切分字幕，或将直接丢弃，毕竟转化太大的视频也是很浪费钱的。
- 以什么形式将文章推送给你，可以是邮件或其他形式，每个人的场景不一定相同。
- 定时触发的能力，我们可以用 crontab 或者 python 的 schedule 库实现。
- 自动生成的文章可能存在一拼写错误，这是由于字幕识别导致的，不过不影响我们阅读。

---

## **总结**

一个简单小工具就实现对 YouTube 频道的自动监控，快速获取视频字幕并生成结构化中文文章，减少不必要的时间浪费，从而提升我们信息获取的效率。

我要做个赖人。



