---
title: "基础插件"
weight: 2
---

# 基础插件

重点来了，接下来我们一起来看看 zsh 的效率神器 - 插件能力吧。本问先给大家推荐 5 款常用的插件。ohmyzsh 提供的所有内置插件，都可以在仓库 [ohmyzsh/ohmyzsh/plugins](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins)  中找到，每个插件都有相应的介绍文档。

本教程将要介绍的 5 个 oh-my-zsh  内置插件，如下所示：

- [git](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)，Git 插件，其实就是提供一些常用的 git 命令别名。
- [web-search](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/web-search)，命令行打开搜索引擎，已支持大部分搜索引擎；
- [jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools)，用于格式化 json 数据；
- [z](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/z)，基于历史访问目录的快速跳转；
- [vi-mode](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode)，使用 vi 模式编辑命令行；

启用所有插件，打开 `zshrc` 配置，把这些内置插件都打开，如下所示：

```bash
plugins=(git web-search jsontools z vi-mode)
```

### 插件 1 - git

Git 插件提供了 git 命令的大量别名，查看[git 插件文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

如下一些常用命令的别名：

```bash
git clone     -> gcl
git status    -> gst
git commit    -> gc
git add       -> ga
git add --all -> gaa
git diff      -> gd
git push      -> gp
git pull      -> gl
```

更多命令的映射关键关系，可自行查看它的[文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git/)。

这个插件不错，但有个缺点，这么多可用别名，我又记不住，岂不是成了摆设。如果想用好，我每次都用去查文档吗？不查文档行不行呢？

当然也是可以的，oh-my-zsh 中启用的一些其他插件可能也会有别名。

其实，有一个插件可帮忙我们解决这个问题，叫做 `you-should-use`，这是下期要介绍的一个插件。简单说下，它的作用是，当我们输入一个命令时，如果这个命令存在别名，它会提示我们要使用别名。

### 插件 2 - web-search

[web-search](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/web-search/) 提供了在终端直接搜索信息的能力。

当然，其实也不是完全在终端完成，它会自动跳转浏览器，转到指定的搜索引擎执行搜索请求。

效果大概就是下面这样：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-04.gif)

常见的搜索引擎基本都是支持的，诸如 google, bing, baidu, 甚至是 github 等。

不过，我也得承认，其实这个插件一般我本人很少用，因为我已经安装了另外一个工具 alfred（替代 mac 默认的 spotlight），我都是通过它直接启动搜索。

### 插件 3 - jsontools

接下的这个插件，名为 [jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools/) ，即用于 json 的 tool。其实它只提供了一些操作 json 的基本命令，如下:

- pp_json 实现 json 字符串格式化；
- is_json 判断是否是 json；

我们直接看下演示效果吧，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-05.gif)

还是得说明，如果你没有更好的方案，安装了 oh-my-zsh，这是个不错的选择，因为你可能以前都没用过这类工具。

不过其实这个插件呢？我也很少用。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-14.gif)

我习惯使用一款叫做 jq 的命令，如果你了解它，就知道它多强大。后面说到高效命令的时候，会介绍到它的。

### 插件 4 - z

[z 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/z) 可用于快速的目录跳转，我觉得大部分人在使用 Linux 都被 cd 跳转目录跳转烦恼过。

z 就是这个烦恼的救星。

想查看更多信息可找 [z 原仓库 - zsh-z](https://github.com/agkozak/zsh-z) 查看。oh-my-zsh 下的 z 文档说明中提到，它是从这个 zsh-z 的插件中拷贝而来的。

我们来介绍它的用法，简单来说，它是基于历史访问过的目录快速跳转。我们无需输入全路径，即可完成目录切换。

下面是一些实际案例。

首先，我直接输入 z，紧跟 tab 键，会看到如下的效果。它会直接将访问过的目录都列出来。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-06.gif)

这些由 tab 产生的自动补全目录都是历史访问过的目录。因为，在没有输入任何内容的情况下，我们输入 tab 的，它列出最近访问过的目录。

如果我们输入形如 z substring，即提供子字符串，它们将所有匹配 substring 的目录都列举出来。

效果如下：

例如，我们输入 z blog，紧跟 tab 键，会直接列出访问过包含 blog 的目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-07.gif)

如果输入内容只有一个关联的目录名，它会如图上一样直接补全。

演示效果：

我们输入 z tmux，因为匹配 tmux 的目录只有一个，将会被直接选中。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-08.gif)

当然，其实这里匹配的目录名只有一个，直接输入 Enter 就可以进入目录，无需 tab 选择多次一举了。

演示效果：

我们输入 z tmux，直接 Enter 确认，即可进入到目录。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-16-zsh-themes-and-plugin-09.gif)

z 非常强大是吧？

其实，有一款更强大的命令，名为 zoxide，也提供了类似的能力，它的灵感是来源于 z。我一般用的是它，后面我会介绍。

当然，这不妨碍你继续使用 oh-my-zsh 内置的工具 z，毕竟它很容易配置。

### 插件 5 - vi-mode

[vi-mode 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode) 支持在命令行开启 vi 模式，利用 vi 键进行命令行编辑。这个插件，视个人情况，是否使用吧。如果你是一个 vi 忠实用户，可考虑开启。否则，还是简单最好，否则容易影响心情。

这个插件就不多介绍了，更多查看 [它的文档](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vi-mode)。另外，如果确实对 vim 感兴趣，也可以考虑另外一个 vi 插件，名为 [zsh-vi-mode](https://github.com/jeffreytse/zsh-vi-mode)，它的能力更强大，也解决这个默认 vi 插件的一些不好用的 bug，不过它的配置有点复杂。


