---
title: "fd - 更易用的文件查找命令"
weight: 2
---

# [fd](https://github.com/sharkdp/fd)

[fd](https://github.com/sharkdp/fd) 是一款文件查找命令，可替换系统默认 find，它的体验更友好，且查询效率极高。我们在使用传统的 find 时，要经常查手册看帮助文档，但使用 fd，它的默认行为就能满足我们大部分的需求。

如何使用呢？

## 安装

```zsh
brew install fd
```

## 递归

文件系统下搜索文件名，最常见的场景是递归搜索文件名包含 pattern 的文件，不知道有你是否能立刻想起来 find 如何写呢？

示例：遍历查找。

```bash
$ fd pattern
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-01.gif)

## 正则

如果要正则查询，pattern 默认即支持正则表达式。

查找文件名包含日期的文件：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-02.gif)

或者查找所有的 go 代码文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-03.gif)

这个表达式更加正确的表述是 fd `.*\.go$`，查出所有以 `.go` 结尾的文件。

## 通配符

fd 同样是支持统配符的，通过 `-g` 选项指定通配符。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-04.gif)

## 文件类型

前面的查找 go 文件，其实也是查找类型是 Go 源码。但正则和通配符，都比较繁琐，可直接通过 `fd -e go pattern` 的模式直接寻找指定扩展的文件。

示例：查找 python 源码文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-05.gif)

如果无需 patern，则表明查找所有的 `.py` 的文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-07.gif)

## 隐藏文件和 `.gitinogre`

fd 搜索效率高的一个原因是它默认不查找隐藏文件和 `gitignore` 文件。

如想启用隐藏文件的查找，可指定 -H 选项，示例如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-06.png)

从上图可知，其中包含了隐藏目录 .git 下的文件。

除了默认忽略 hidden 文件，fd 还会忽略 .gitignore 中的文件，选项 -I 即可查找包含在 `.gitinogre/.fdignore/.ignore` 的文件。

```bash
$ fd -I pattern
```

## 其他示例

fd 的更多选项，我就不一一介绍了，可以查看如下表格。

案例                    | 命令                          | 说明
----------------------- | ----------------------------- | -------
忽略大小写              | fd -i readme.md               | 包含 readme.md 和 README.md
大小写的 smartcase 模式 | fd -s readme.md               | 包含 readme.md 和 README.md
---                     | fd -s README.md               | 只包含 README.md
查找结果显示详细信息    | fd -l .go                     | 查询结果显示文件详情
大小过滤                | fd -S +1000k                  | 查询大小 > 1000k 的文件
仅查找目录              | fd --type/-t directory golang | 查询所有目录文件
查询并执行命令          | fd --type/-t file -x wc -l    | 查询文件并计算行数
---                     | fd --type/-t file -X vim      | 查询所有文件并用 vim 打开

## 性能

除了忽略隐藏和 gitignore 中的文件外，fd 在性能上也是碾压 find。通过 hyperfine 命令测试 fd 与 find。

```zsh
hyperfine --warmup 3 "find . -iname *.go" "fd -i -g '*.go' -uu"
```

性能对比结果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-08.gif)

压测结果显示，fd 相较于 find 快了 3.88 倍。

