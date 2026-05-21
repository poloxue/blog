---
title: "基础插件 2"
weight: 3
---

本文是基础篇插件的第二篇，继续介绍 4 个常用的 zsh 插件。我将涉及的插件，如下所示：

name       | description
-----------| ---------------
copypath   | 拷贝路径到剪贴版；
copyfile   | 拷贝文件内容到剪贴板；
copybuffer | 拷贝命令行内容到剪贴板；
sudo       | 命令行开头快速添加 sudo；

让我们正式开始。

## 推荐插件

先说 oh-my-zsh 的内置插件。

打开 zsh 配置文件 ~/.zshrc，将要使用的 oh-my-zsh 的内置插件提前配置。

```bash
plugins=(... copypath copyfile copybuffer sudo ...)
```

保存退出，执行 `source ~/.zshrc 生效`。

### copypath

[copypath](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copypath) 的用途如其名，就是用来 copy 路径的。

支持两种用法。

copypath: 无参数，直接拷贝当前路径；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-01.gif)

copypath <文件或目录>：拷贝指定文件或目录的绝对路径；

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-02.gif)

相比于 `pwd` 之后再拷贝，这种方式真的是省心省力的方式。

### copyfile

[copyfile](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copyfile)，用于拷贝文件内容，命令格式 copyfile <文件路径>。

假设，现有一个文件 test.txt。

```bash
cat test.txt
Hello oh my zsh
```

一个测试命令，`copyfile test.txt`，即可将 `test.txt` 文件中的内容拷贝到剪贴板中。

效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-03.gif)

无需鼠标选中复制粘贴。

### copybuffer

[copybuffer](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copybuffer)，是用于快速复制当前命令行的输入。

如何使用呢？

它不同于前面两个快捷键，要通过 CTRL+o 快捷键拷贝命令行内容。

特别说明，我在测试的时候，发现 copybuffer 与 vi-mode 存在冲突，不过如果启用了 vi-mode， 命令行内容拷贝可直接使用 yy，无续开启 copybuffer；

### sudo 

sudo 的主要作用是，当我们输入某个命令，如 vim /etc/zshrc，发现没有系统权限，利用 sudo 插件，可快速将 sudo 作为前缀添加到命令最前面。

演示效果如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-19-zsh-6-powerful-plugins-06.gif)


