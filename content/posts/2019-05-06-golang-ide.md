---
title: "Go 的那些 IDE"
date: 2019-05-06T12:28:00+08:00
draft: false
comment: true
tags: ["Golang"]
---

经过前面的一系列工作后，Go 的语言环境已经搭建完成。

我们初步体验了 Go 提供的大部分命令。但在正式进入开发之前，还有件工作要做，那就是选择一款适合自己的 IDE。

## 为什么使用IDE

"程序员为什么要使用 IDE"，在一些社区论坛，经常可以看到这样的提问。关于是否应该使用IDE，每个人都有着自己的看法。

早期，程序的开发并不需要 IDE，那是以机器码编程为主的时代。后来随着计算机行业发展，为了进一步提升工程开发效率，IDE就产生了。

要明白的是，IDE主要是通过把各类命令工具集整合起来，开发的一套易于程序开发的软件，通常它帮我们形成一套高效的编程开发习惯。最终目标是为了提升项目的开发效率。

了解了 IDE 的本质，如果喜欢折腾，我们完全可以把诸如 vim 或 emacs 等文本编辑器打造一款属于自己的IDE。

## 支持哪些功能

无论用的是市面上已有的 IDE，还是 vim 纯手动打造的IDE，都离不开一个话题：IDE 涉及的功能有哪些？文本编辑的能力就不必介绍了，它是最基本的功能。

### 快捷键

双手不离键盘是高效开发中非常重要的一点，要做到它，我们就需要依赖功能强大的快捷键。IDE 通常都有一套独有的快捷键规范。当习惯了一款 IDE，快捷键或许是大家轻易不愿更换 IDE 的重要原因之一。

### 代码高亮

代码高亮主要涉及变量、函数定义、类、常量、特殊符号、关键词等。代码高亮可以提高代码阅读体验，对不同语法采用不同的配色方案，也可降低代码错误的发生几率。而且，IDE一般都支持自定义配色，可以由个人爱好自由设置。

### 代码格式化

为了方便团队开发，在项目开发前，通常都会制定统一的代码规范。制定好的规范需要遵从，而 IDE 一般都支持代码的格式化功能，帮我们更方便地实现目标。需要说明的是，不同于 Go，很多编程语言并没有类似 gofmt 的命令，代码规范也是多样。

### 代码提示

IDE的代码提示能根据输入快速给出一系列的建议列表，比如参数信息、成员列表、代码片段等。为了给出更精准的提示，一些IDE可能甚至会分析用户历史的操作记录。感觉这俨然已经是一个小型的推荐系统了。

### 导航跳转

大型项目的代码量通常较大，涉及文件也较多。在开发时，我们经常需要在变量、函数、类等代码间跳转。最不便利的方式，我们可以通过键盘方向键或鼠标实现切换。IDE通常都实现了在变量、类型定义、函数定义、文件之间快速跳转的方法。

### 代码调试

多数情况下，通过打印函数就可以实现代码调试。但通过系统化工具提供的调试功能，我们就能应付各种复杂的场景。调试工具通常支持各种断点调试能力、变量观察等功能。

### 构建编译

Linux 下最常用的构建工具应该是 Makefile，之前开发C/C++用的便是make。但有些语言项目用它构建会很复杂，比如 Java。IDE 的构建编译功能可以快捷地生成目标文件。编译功能通常使用的是语言自带编译器，比如 Go 用 go build 命令。

### 其他功能

当然，除上面介绍的这些，IDE可能还有很多其他能力，比如代码重构、文件历史记录、语言环境管理、数据库管理等。只要是能想到的功能，基本都可集成进来，现在的 IDE 俨然已经完全超出了传统IDE的范畴。

## GO有哪些IDE

GO的发展已有十几年之久。在这期间出现了很多能编写 GO 语言的 IDE，把它们都详细介绍一遍是不现实的。接下来，重点介绍我比较了解几款IDE。

### Goland

Goland，商业公司 jetbrains 推出的 Go 集成开发环境，它真的是无比强大。

我相信很多程序都用过他们家的IDE，比如Java的 Intellj IDEA、PHP的PHPStorm、Python的PyCharm、C++的CLion、前端的WebStorm等。使用JetBrains的IDE，我们可以享受到它优秀的开箱即用的体验和 jetbrains 积累十几年的插件体系。

前些年，也就是Goland发布之前，如果我们希望用jetbrain的IDE进行GO的开发，需要通过它提供的插件支持。Goland发布后，这些插件似乎已经下架了。

不得不承认，Goland的功能层面做的确实非常完美。不过有几点我想吐槽一下，首先必须要提的是，Jetbrians的IDE基本都存在着卡顿的毛病，资源消耗比较严重。虽然一些大牛提供了优化方案，但体验下来，和其他IDE依然没有相比。

Goland 开箱即用，它的问题很少，确实没有多少可介绍的，安装完成基本就可以开始编码！

### VS Code

由微软开发的一款功能强大的现代化轻量级代码编辑器IDE，免费开源。通过它强大的插件扩展能力，VS Code几乎支持主流语言的项目开发。毫无例外，GO也是其中之一。

我之所以尝试VS Code，并非所谓的极客思维，喜欢瞎折腾。而是因为jetbrains的IDE经常会卡的心痛，而且自己经常会在不同语言间切换。一次启动多款Jetbrains的IDE，这是很痛苦的。

为VS Code加入GO的开发能力，要安装一款插件即可，可参考 [VsCode 的 Golang 文档](https://code.visualstudio.com/docs/languages/go)。

安装时，可能遇到一些问题，常见的就是，在安装一些依赖包时会出现网络下载失败。关于原因就不说了，大家都明白。不过，问题还是要解决的。举个具体的例子吧！在GO插件时，我们会通过go get golang.org/x/tools/xxx安装某个包，这时候大概率出现网络连接错误。我们可以通从github找到对应的仓库，golang/tools，然后使用git命令下载后，放在GOPATH指定的目录下，然后再安装即可。

插个题外话，VS Code使用的是Electron开发的，Electron是用HTML，CSS和JavaScript来构建跨平台桌面应用程序的一个开源库，NodeJS与Chromium的结合。因此，利用浏览器的特性，利用VS Code，我们能实现很多奇葩的插件。

[GitHub Daily：装上这几个 VSCode 插件后，上班划水摸鱼不是梦](https://zhuanlan.zhihu.com/p/58302580)

### Vim GO

细究起来，vim 是一款文本编辑器，但它却拥有了很多不该属于文本编辑器的能力，比如单词补全、ctags标签跳转、窗口分隔、崩溃文件恢复、文件diff、400多种文本高亮等。最重要的一点是，vim有一套自己的脚本语言，这为它通过插件扩展自己的能力提供了可能。

将vim扩展成一款适合自己使用的GO IDE，不仅要编写许多复杂的配置与脚本，还需要各种插件的相互配合，才能实现我们的目标。比如前面介绍的那些IDE的常见功能，在vim中都要逐一配置实现。

GO的vim环境搭建，需要用到一款非常重要的插件，vim-go，youtube上还有他的[分享视频](https://www.youtube.com/watch?v=7BqJ8dzygtU)，有兴趣可以去看看。

vim-go提供了诸如代码的编译、执行、测试、代码重构、错误提示等各种功能，具体了解可查看 [vim-go教程](https://github.com/fatih/vim-go-tutorial)。

说明一点，虽然 vim 支持插件扩展，但它要集成出 VS Code 的体验还是非常困难的。

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-05-06-golang-ide-01.png)

当前我用的主要就是这三款IDE，Goland VSC 和 vim。当然，还有很多其他IDE，下面也简单介绍下，但因为没怎么使用过，所以很难有经验之谈了。

最近将 vim 迁移到了 Neovim，体验下来，真的是一级棒。

### Sublime Text

最初用VS Code，感觉它的使用习惯和Sublime相似。但说到Sublime，都说它是强大文本编辑器，而它的编码能力也是插件扩展来的。GoSublime就是为Sublime扩展GO功能的插件。

### LiteIDE

一款轻量级的IDE，听说是由中国人开发的。可能在Goland出现之前比较流行。也或许是自己孤陋寡闻，不知道现在还有多少人在用。

### Eclipse

开源的IDE，盛行了多年，有着丰富的资源和粉丝人群，应该是Java开发最喜欢的IDE吧。GoEclise是Eclipse针对Goland的插件。从github了解到，这个项目好像很久没有更新了。

### Atom

与VS Code一样，都是基于Node-Webkit，即Electron，开发的。是由github开源的文本编辑器。go-plus是Atom针对Golang开发的插件

## 总结

本篇文章从为什么要使用IDE谈起，介绍了IDE的一些发展史。同时，总结了一款基本的IDE通常都会提供哪些功能。只要了解了这些，可以帮助我们以后更好地使用它们。最后，介绍了现在市面上流行的几款IDE，并在我力所能及的范围内分析了它们各自的优劣。
