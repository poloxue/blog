---
title: "快速安装"
weight: 2
---
## 安装

对于不同系统，zsh 的安装命令，如下所示： 

Debian
```bash
apt install zsh
```

Centos
```bash
yum install -y zsh
```

Arch Linux
```bash
pacman -S zsh
```

Fedora
```bash
dnf install zsh
```

对于 macOS 系统的用户，MacOS 的默认 shell 从 2019 开始以前替换为 zsh，该步骤可省略。可阅读：[What is Zsh? Should You Use it?](https://linuxhandbook.com/why-zsh/#:~:text=Zsh%20is%20more%20powerful%20and,more%20advanced%20features%20shipped%20in.) 其中有介绍为什么 2019 macOS 将默认的 shell 从 bash 切换到 zsh。

我看下来，主要原因就是版权问题啦。

如果你是个老古董，还是用 MacOS 2019 之前的系统，可通过如下命令安装：

```bash
brew install zsh
```

安装完成后，将 zsh 设置为默认 shell，命令如下所示：

```bash
chsh -s /bin/zsh
```

通过如下命令检查下是否成功。

```bash
echo $SHELL
zsh
```

