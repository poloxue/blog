---
title: "zoxide"
weight: 2
---

# zoxide

在正式介绍 [zoxide](https://github.com/ajeetdsouza/zoxide) 前，尝试提前问自己一个问题，Linux 默认命令 cd 好不好用？我的答案是，相当难用，无论多么丝滑的操作，一旦遇到 cd，只能说一句 f**k。

你是不是经常这样使用 cd 呢？

```
cd ../
cd ../
cd ../
cd ../../../
cd x/
cd y/
cd z/
```

如果不想被 cd 折磨的话，我强烈推荐这个工具：[zoxide](https://github.com/ajeetdsouza/zoxide)。

[zoxide](https://github.com/ajeetdsouza/zoxide) 是一款受到 z 和 autojump 启发而来的命令，它会记录访问过的目录，通过搜索找到最匹配你目标的目标。从而实现以 **最最最最** 少按键就能实现目录跳转。

一般情况下，我们关注的目录就那几个，90% 的情况用它的快速跳转能力即可，而一些特殊情况，cd 绝对路径即可，亦或者是使用它提供的另一种方式，交互式搜索。

前面我写过一篇文章 介绍了 oh-my-zsh 提供的 z 插件，zoxide 与 z 相比更易于使用。这有一份对比报告：[zoxide vs zsh-z](https://www.libhunt.com/compare-zsh-z-vs-zoxide)。

具体介绍它的安装使用吧。

### 安装

```zsh
brew install zoxide # 其他系统请查看 GithHub README.md
```

### 配置

在 zsh 中使用 zoxide 要简单配置下，一行命令将 zoxide 初始化命令追加到 `~/.zshrc` 中。

如下所示：

```bash
echo 'eval "$(zoxide init zsh --cmd z)"' >> ~/.zshrc
```

生效后，即可通过 z 命令使用 zoxide 的能力。

亦或者，如果觉得要从习惯于使用 cd 切换到 z 有难度，可直接将 z 命名为 cd，直接替换掉系统的 cd 命令，配置 `--cmd cd` 即可。
```bash
echo 'eval "$(zoxide init zsh --cmd cd)"' >> ~/.zshrc
```

我将会使用 zoxide 直接替换 cd 命令进行演示，即第二种配置方式。

### 使用

正式开始使用前，先假设我有如下目录结构：

```
~/Hello
 |_ ./golang-examples
 |_ ./python-examples
 |_ ./rust-examples
 |_ ./trading-strategies
```

由于 zoxide 是在历史访问路径的基础上智能选择。为了便于演示，我把之前的访问历史记录已经清空了，并通过如下命令初始化访问历史：

```zsh
cd ~/Hello/golang-examples
cd ~/Hello/python-examples
cd ~/Hello/rust-examples
cd ~/Hello/trading-strategies
```

假设现在在 `~` 用户目录下，如下是演示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-05.gif)

主要演示了 zoxide 的三个能力，分别是全名匹配、部分匹配和如果存在重名目录下会选择智能最优的目录。

首先是全名匹配，通过 zoxide 的 cd 命令，在 `~` 目录下快速跳转到 golang-examples 目录，直接输入 `cd golang-examples`。

其次是部分匹配，由于记录中只有一个目录包含 python，执行 cd python 会直接进入到 `python-examples`；

最后是重名按算法选择最优目录，我不清楚它的具体算法，但观察来看，它当前在 python-examples 目录下，执行 cd examples 会优先找到最近访问的包含 examples 的目录，且非当前位置，即 `golang-examples`。

如果觉得它的智能算法不够灵活，还可以尽量补全路径，和使用普通 cd 一样。如果还想要它的效率，也可以进入它的交互搜索模式，zoxide 支持两种方式进入交互模式：

一种是输入 cd + 目标名称 + 快捷键 \<Space\>+\<Tab\>，进入交互选择模式，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-07.gif)

另一种是直接使用 cdi 命令也可进入交互搜索模式，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-06.gif)

补充，zoxide 本身也是个命令，你可以用它增删改查，管理历史方案记录。

```bash
$ zoxide query golang # 返回最匹配 golang 的目录
$ zoxide query golang --list # 返回所有匹配 golang 的目录
$ zoxide -h # 更多帮助信息
zoxide 0.9.2
Ajeet D'Souza <98ajeet@gmail.com>
https://github.com/ajeetdsouza/zoxide
一个更智能的终端 `cd` 命令

使用方法：
  zoxide <命令>

命令：
  add     添加一个新目录或增加其排名
  edit    编辑数据库
  import  从另一个应用导入条目
  init    生成 shell 配置
  query   在数据库中搜索目录
  remove  从数据库中移除目录
```

差不多，zoxide 大概就介绍的这么多吧。希望对于想用它替换掉 cd 的朋友有所帮助吧。


