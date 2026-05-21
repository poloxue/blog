---
title: "搜索查找"
weight: 2
bookCollapseSection: true
---

# 搜索查找

本文将介绍三个高效搜索命令，分别是 fd、ripgrep 与 fzf。fd 和 ripgreap 对标的是传统 grep 和 find，它们在性能和体验上有大幅提升。

如果你成为 10x 程序员，强烈推荐使用它们。

快速一览：

- **[fd](https://github.com/sharkdp/fd)**，目录与文件搜索命令，比默认 find 更易于使用，而且查找速度上更快；
- **[ripgrep](https://github.com/BurntSushi/ripgrep)**，可用于高效的内容搜索，比默认的 grep 命令速度更快；
- **[fzf](https://github.com/junegunn/fzf)**，命令行交互式模糊搜索工具，可与其他命令进行结合，提高使用体验；

如上的 fd 与 ripgrep 是由 rust 编写，性能上完虐传统的 find 与 grep。

