---
title: "什么是 zsh？我是否应该使用 zsh"
date: 2023-09-18T14:06:02+08:00
draft: false
comment: true
tags: ["bash", "zsh", "shell"]
---

我们知道，在所有的 Linux/Unix 中 shell，Bash 是最流行的，它是多数 Linux 发行版的默认 shell。除了 bazh，zsh 是另外一款非常流行的 shell。它功能更强大，而且还是 macOS 中的默认 Shell。

zsh 为什么如此受欢迎？我们是否应该使用它呢？


## 什么是 zsh？

“Z shell” 最初是由 Paul Falstad 在普林斯顿大学就读时开发。

Zsh 整合了绝大多数主流 Shell 中的功能，如 Bourne-Again Shell (Bash)、Korn Shell (ksh)、C-shell (csh) 和 tcsh。故而，zsh 与这些主流 shell 都有一定程度的兼容，是其更受用户欢迎。

如今，Zsh 俨然已经是一个庞大的开源项目（非 Paul Falstad 维护），拥有一个有大量用户和贡献者的社区。而且，自 2019 年以来，它成为了 Apple macOS 的默认 Shell。

## bash vs zsh

这两个项目都还在积极开发中。这使它们在功能上越发接近，但差异不可能完全消除。默认，zsh 更强大且更容易自定义，而某些功能， Bash 需要一些额外的脚本（插件）才能实现。

zsh 优于 Bash 的主要功能是：

zsh 的补全能力强大，bash 的 Tab 补全是从头匹配，如 mn 匹配 mnt，而非 findmn，而 zsh 可同时匹配 mnt 和 findmn； 

zsh 的命令行历史是在 terminal 间共享，结合自动补全，进一步增强了用户体验；

![](https://linuxhandbook.com/content/images/2022/02/zsh-tab-completion.png)

zsh 还提供自动纠错能力，如果你输入太快，它能智能给你一个可能正确的建议；

zsh 的配置能力更强，支持构建更精美的提示主题；

zsh 参数的扩展能力比 Bash 更强大；

zsh 有大量插件、主题、框架，如 oh-my-zsh 框架，能助你快速配置出一个强大终端；

## 我们是否要用 zsh？

zsh 已被证明是一个功能强大高效、高可定制的 shell，它使我们能轻松拥有一个体验丝滑的 CLI 终端。

如果你一个时常与命令行为伴的开发人员，那 zsh 绝对是一个不错选择。再者，如果你愿意投入更多时间去探索它，将会发现更多惊喜。

## 我该放弃 bash 吗？

bash 不会消失，这是事实！它是大多数 Linux 发行版中的标准 shell，这意味着你在世界上的大多数 server、container、vps 和云实例都能找到它。

另外，对于 shell 脚本，除非有一些特殊需求，否则最好编写标准的 bash script。

你可能主要使用 Zsh，但不要认为你不会再接触 bash，至少，bash 脚本在未来的几年仍是一个可靠的选择。

## 为什么 zsh 是 macOS 的默认 shell？

这个问题的答案，除了与 zsh 的强大特性有关，还一个重要因素是关于 License。

切换到 zsh 前，MacOS 多年以来内置的一直是过时版本的 bash （v3.2, 2007 年发布），作为默认 shell，这是 bash 以 GPLv2 授权的最后一个版本。 3.2 之后，新版本的 Bash 均是 GPLv3，这一许可对于 Apple 似乎无法接受。

终于在 2019 年，Apple 转向了 MIT License 的 zsh。

## 结论

zsh 能增强你的 CLI 终端使用体验，而且，只需简单安装即可使其大放异彩。

但，不得不说，bash 现在依然是 shell 脚本的不错选择，熟悉标准 bash 安装对还是会非常有用。它涉及 DevOps、系统管理、云计算和容器，因为大多数 Linux 发行版的默认 shell 还是 bash。

请安装开始你的探索吧。我相信，你会发现它的强大。或许，最终它会成为你的默认 shell。

博文地址：[什么是 zsh？我是否应该使用 zsh](https://www.poloxue.com/posts/2023-09-16-what-how-to-use-zsh/)，译：[What is Zsh? Should I Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in)

