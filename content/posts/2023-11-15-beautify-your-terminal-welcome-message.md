---
title: "我的终端环境：终端启动消息 - ASCII art"
date: 2023-11-13T18:38:48+08:00
draft: false
comment: true
description: "本文介绍如何配置终端的 welcome 消息，让你每天都是好心情。"
---

{{< video bb_id="280934228" yt_id="T1W1wYQhB64" >}}

本文介绍如何设置 MacOS 系统的终端启动消息，或者说欢迎消息。

> 本文介绍的内容同样适用于其他类 Unix 系统。
>
> 某种意义上，这是一个无用的小知识，但它确实很有趣。毕竟，不是任何事情都要追求所谓价值，有趣也挺重要的。

## 登录消息

每天打开 terminal 终端，系统默认会打印一串的消息，如 "Last Login xxx" 之类的消息。是否想过让这个默认消息更加丰富一些？

如 MacOS 这样的类 Unix 系统默认有两种方式，一种是基于系统的 motd，另一种是通过启动脚本打印消息。

> 系列阅读：
>
> - [我的终端环境：iTerm2 的安装与体验](https://www.poloxue.com/posts/2023-09-25-install-iterm2-as-my-developing-environment/)
> - [我的终端环境：zsh 安装与主题，推荐 7 个提升效率的 zsh 插件](https://poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
> - [我的终端环境：6 个强大的 zsh 插件](https://www.poloxue.com/posts/2023-10-19-zsh-6-powerful-plugins/)
> - [我的终端环境：与众不同的 zsh 主题 - powerlevel10k](https://www.poloxue.com/posts/2023-10-20-zsh-theme-powerlevel10k/)
> - [我的终端环境：高效 shell 命令（一）之目录文件命令 - exa、zoxide 与 bat](https://www.poloxue.com/posts/2023-10-28-high-productivity-shell-commands-part1/)
> - [我的终端环境：高效 shell 命令（二）之高效查找与搜索 - fd ripgrep fzf](https://www.poloxue.com/posts/2023-10-30-high-productivity-shell-commands-part2/)
> - [我的终端环境：高效 shell 命令（三）之提效 web 开发 - entr httpie jq](https://www.poloxue.com/posts/2023-11-02-high-productivity-shell-commands-part3/)
> - [我的终端环境：高效 shell 命令（四）之20+1 个 modern-unix 命令](https://www.poloxue.com/posts/2023-11-07-high-productivity-shell-commands-part4/)
> - [我的终端环境：终端启动消息 - ASCII art](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
> - [我的终端环境：终端启动消息 - pfetch/neofetch/fastfetch](https://www.poloxue.com/posts/2023-11-16-beautify-your-terminal-welcome-using-fetch/)
>
> 更多待续...

### motd

首先，通过 motd 的实现方案。motd，即 message of today，类 Unix 系统基本都支持，我们只要将启动消息写入到 `/etc/motd`中，即可定制终端启动消息。

对于公用服务器，系统管理员用它可向普通用户发送消息。而且，有部分 Linux 发行版甚至用它的默认配置给用户发送广告。

演示看效果吧。

将文字 "Hello World. Have a nice day!" 写入 `/etc/motd` 中，开启一个新的终端。

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-01.png" >}}

motd 的缺点是它是静态内容，如果要更新内容，则需编辑文件。

motd 的动态效果实现方案，可基于 crontab 定时更新 motd 中的内容实现 motd 伪动态。不过，这种方式主要是适合于不停机的服务器，对于个人电脑，经常处于关机状态，可能不太适合。

### 启动脚本

实现真正意义上的动态 motd 消息，可直接通过系统启动脚本打印环境消息，如在 `~/.zshrc` 中添加欢迎消息打印脚本，消息内容可任意定制。

如 MacOS 系统在启动时，通过解析 `uname -a` 输出内容，打印系统信息和内核版本：

```zsh
echo `uname -a | cut -d' ' -f1 -f3`
```

加入到 `~/.zshrc` 文件中，效果如下所示：

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-02.png" >}}

到这里，终端欢迎消息定制的基本内容介绍结束。但问题是，这明显不够惊艳，对于这种无用小知识，肯定要 "华而不实"，但现在没有看到任何惊艳之处。

To be continued...

## ASCII art 定制欢迎消息

文字单一枯燥，如何给欢迎消息引入不一样的内容？我们将引入 ASCII art 改造枯燥的启动罅隙。

所谓 ASCII art，是通过 ASCII 字符表达图片的一种方式。在文字比图像更稳定的场合，如终端、或者早起没有图像显示能力的设备，ASCII art 显然是更好的选择。

示例如下：

```plain
 _          _ _                            _     _
| |__   ___| | | ___   __      _____  _ __| | __| |
| '_ \ / _ \ | |/ _ \  \ \ /\ / / _ \| '__| |/ _` |
| | | |  __/ | | (_) |  \ V  V / (_) | |  | | (_| |
|_| |_|\___|_|_|\___/    \_/\_/ \___/|_|  |_|\__,_|
```

```plain
 __________________
< have a nice day! >
 ------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-03.png" >}}

如何生成 ASCII art？

ASCII art 一般有 ASCII text 和 ASCII image 两种形式。可通过在线站点生成，或通过一些命令生成。

### 在线网站

推荐两个工具站点，[ASCII Generator](https://ascii-generator.site/) 和 [ASCII banner](https://manytools.org/hacker-tools/ascii-banner/)。

演示效果，使用 ASCII Generator 生成 ASCII 3D 文字，如下所示：

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-04.png" >}}

如想找一些现成的 ASCII art，查看 [ascii-art](http://ascii-art.de/)，其中有一些现成的可供选择。

### 命令工具

举一反三，工具站点能生成 ASCII art，那必然通过命令也可以做到，必须都是程序实现。

MacOS 一键安装：

```zsh
brew install figlet cowsay fortune lolcat 
```

**figlet 生成 ASCII text**

figlet，可用于 ASCII text。它常用于开源项目生成 text banner。

使用演示：

```plain
> figlet hell world
 _          _ _                            _     _
| |__   ___| | | ___   __      _____  _ __| | __| |
| '_ \ / _ \ | |/ _ \  \ \ /\ / / _ \| '__| |/ _` |
| | | |  __/ | | (_) |  \ V  V / (_) | |  | | (_| |
|_| |_|\___|_|_|\___/    \_/\_/ \___/|_|  |_|\__,_|
```


**fortune + cowsay 牛说名言**

对于 ASCII art 内容，还有常用的一个组合命令：fortune + cowsay。

fortune 用于生成随机生成 "英文名言"。

```zsh
❯ fortune
Things will get better despite our efforts to improve them.
                -- Will Rogers
```

如果希望有中文内容，可安装阮一峰老师提供的 [fortune 中文数据库](https://github.com/ruanyf/fortunes)。

cowsay，即 "牛说" 的意思，配合 fortune，会生成一个 "牛说某个格言" 效果。

```
❯ fortune | cowsay
 ________________________________________
/ Shame is an improper emotion invented  \
| by pietists to oppress the human race. |
|                                        |
| -- Robert Preston, Toddy,              |
\ "Victor/Victoria"                      /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

默认的 "牛" 的形象是可修改的，如换成 "羊"，通过 `-f` 修改。

命令如下：

```plain
❯ fortune | cowsay -f sheep
 ________________________________________
/ Winny and I lived in a house that ran  \
| on static electricity... If you wanted |
| to run the blender, you had to rub     |
| balloons on your head... if you wanted |
| to cook, you had to pull off a sweater |
| real quick...                          |
|                                        |
\ -- Steven Wright                       /
 ----------------------------------------
  \
   \
       __
      UooU\.'@@@@@@`.
      \__/(@@@@@@@@@@)
           (@@@@@@@@)
           `YY~~~~YY'
            ||    ||
```

要查看支持的所有形象，通过如下命令：

```zsh
> cowsay -l
Cow files in /usr/local/Cellar/cowsay/3.04_1/share/cows:
beavis.zen blowfish bong bud-frogs bunny cheese cower daemon default dragon
dragon-and-cow elephant elephant-in-snake eyes flaming-sheep ghostbusters
head-in hellokitty kiss kitty koala kosh luke-koala meow milk moofasa moose
mutilated ren satanic sheep skeleton small stegosaurus stimpy supermilker
surgery three-eyes turkey turtle tux udder vader vader-koala www
```

如果想定制更多形象，可参考上面目录 `/usr/local/Cellar/cowsay/3.04_1/share/cows` 中的案例，通过 `-f` 执行也是可以的。

**lolcat - 随机彩虹着色**

lolcat，可用于给输出进行随机的彩虹着色，如给以上输出 ASCII 文字着色，可通过 lolcat 实现随机彩虹效果，效果如下：

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-05.png" >}}

或给 cowsay 的输出着色，效果如下：

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-06.png" >}}

**程序实现**

如果想通过编程实现文字转 ASCII 或图片也是可以做到的。当然，其实也没啥技术含量，依赖的 Python 的强大的三方库支持。

文字转 ASCII，可通过 pyfiglet 库实现，而图片可以来 pillow 库实现。

```zsh
pip install pyfiglet
pip install pillow
pip install click
```

完整代码的 gist 地址：[ascii-generator](https://gist.github.com/poloxue/b55687913b65f4207fc41150d8330807)。粘贴一份完整代码，如下所示：

```python
import sys

import click

import pyfiglet
import PIL.Image


def image_to_ascii(path):
    try:
        img = PIL.Image.open(path)
    except:
        print(path, "Unable to find image")
        exit(1)

    width, height = img.size
    aspect_ratio = height / width
    new_width = 50
    new_height = aspect_ratio * new_width * 0.55
    img = img.resize((new_width, int(new_height)))

    img = img.convert("L")

    chars = [" ", "J", "D", "%", "*", "P", "+", "Y", "$", ",", " "]

    pixels = img.getdata()
    new_pixels = [chars[pixel // 25] for pixel in pixels]
    new_pixels = "".join(new_pixels)
    new_pixels_count = len(new_pixels)

    ascii_image = [
        new_pixels[index : index + new_width]
        for index in range(0, new_pixels_count, new_width)
    ]
    return "\n".join(ascii_image)


def text_to_ascii(text):
    return pyfiglet.figlet_format(text)


@click.command()
@click.option(
    "--type",
    "type_",
    default="text",
    help="source type, unavailable options: text, image",
)
@click.option("--source", help="text or image path")
def main(type_, source):
    if type_ == "text":
        ascii = text_to_ascii(source)
    elif type_ == "image":
        ascii = image_to_ascii(source)
    else:
        return

    print(ascii)


if __name__ == "__main__":
    main()
```

其中的参数如宽度还有字符，都是可调节的，使用起来也都非常简单。

文字转 ASCII：

```
> python ascii_image.py --type text --source "hello world"
 _          _ _                            _     _ 
| |__   ___| | | ___   __      _____  _ __| | __| |
| '_ \ / _ \ | |/ _ \  \ \ /\ / / _ \| '__| |/ _` |
| | | |  __/ | | (_) |  \ V  V / (_) | |  | | (_| |
|_| |_|\___|_|_|\___/    \_/\_/ \___/|_|  |_|\__,_|
```

图片转 ASCII：

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-07.png" >}}

## 完成设置

静态内容，现在只要将生成的 ASCII art 写入到 motd 文件。如果希望静态文本中的输出保持颜色，可通过 `lolcat -f` 将颜色信息也导入到文本中。

动态内容，将类似于 `fortune | cowsay | locat` 加入到启动脚本 `~/.zshrc` 即可。

到此，初步大功搞成。

## 总结

通过本文，我的奇怪知识又增加了。对了，文中的 ASCII image 的原图是我的博客头像。

最后，下期介绍如何在欢迎消息中展示更丰富的信息，如：

{{< image "2023-11/2023-11-15-beautify-your-terminal-welcome-message-08.png" >}}

我的博文：[我的终端环境：无用小知识 - 终端启动消息配置](https://www.poloxue.com/posts/2023-11-15-beautify-your-terminal-welcome-message)
