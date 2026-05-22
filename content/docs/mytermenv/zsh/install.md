---
date: 2024-02-04
title: "快速安装"
weight: 1
---

# 安装

对于不同系统，zsh 的安装命令，如下所示：

**macOS：**

macOS 从 Catalina 开始默认 shell 就是 zsh 了，无需额外安装。如果你的系统版本较旧：

```bash
brew install zsh
```

**Debian/Ubuntu：**

```bash
sudo apt install zsh
```

**CentOS/RHEL/Fedora：**

```bash
sudo dnf install zsh    # Fedora
sudo yum install zsh     # CentOS 7
```

## 设为默认 Shell

安装完成后，将其设为默认登录 shell：

```bash
chsh -s $(which zsh)
```

重新打开终端，你就进入了 zsh 的世界。第一次启动 zsh 时，会看到初始配置向导——按 `q` 跳过，我们后面通过 oh-my-zsh 统一管理配置。

验证是否切换成功：

```bash
echo $SHELL
# 输出应该是 /bin/zsh 或 /usr/bin/zsh

echo $ZSH_VERSION
# 输出版本号，如 5.9
```

## 基础配置

zsh 的配置文件是 `~/.zshrc`，所有的别名、函数、插件配置都在这个文件里。如果你是从 bash 迁移过来的，可以把 `~/.bashrc` 里的配置搬过来，大部分语法兼容。

不过别急着手动配置——下一章介绍的 oh-my-zsh 会让配置管理变得极其简单。
