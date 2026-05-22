---
date: 2024-04-14
title: "系统信息工具"
weight: 2
---

# 系统信息工具

在终端启动时显示系统信息，既能了解当前环境状态，也让终端更有"个性"。社区中有几款流行的系统信息展示工具，各有特色。

## neofetch

[neofetch](https://github.com/dylanaraps/neofetch) 是最经典的系统信息工具，它以 ASCII 艺术图 + 系统信息的形式展示。不过 neofetch 已于 2020 年停止维护，原作者建议使用其他替代品。

### 安装

```bash
# macOS
brew install neofetch

# Debian/Ubuntu
sudo apt install neofetch
```

### 使用

```bash
neofetch
```

输出示例：

```
          ██████████████          poloxue@macbook
        ████████████████          -----------------
       ██╣  █████  ██             OS: macOS 14.3 (Sonoma)
      ███╣  ██   ║████           Host: MacBook Pro M3 Pro
      ███╣  ║    █████           Kernel: Darwin 23.2.0
     ████║       ██████          Uptime: 3d 12h 34m
     █╣ ██║     ║██║ ██          Packages: 342 (brew)
     █╣  ║█║ ║ ██║  ██          Shell: zsh 5.9
     ████████████████████        Resolution: 3456x2234
       ████████████████          Terminal: iTerm2 3.5
```

### 自定义

neofetch 的配置文件在 `~/.config/neofetch/config.conf`。你可以自定义：

- 显示哪些信息（关闭不关心的项）
- ASCII 艺术图的样式和颜色
- 信息的排列顺序

例如，只显示最关键的信息：

```bash
# ~/.config/neofetch/config.conf
print_info() {
  info title
  info "OS" distro
  info "Kernel" kernel
  info "Uptime" uptime
  info "Packages" packages
  info "Shell" shell
}
```

## fastfetch

[fastfetch](https://github.com/fastfetch-cli/fastfetch) 是 neofetch 的 Rust 重写版，速度和兼容性都有显著提升。如果你觉得 neofetch 启动太慢，fastfetch 是绝佳替代品。

### 安装

```bash
# macOS
brew install fastfetch

# 其他系统
# 从 GitHub Releases 下载二进制
```

### 使用

```bash
fastfetch
```

fastfetch 默认输出比 neofetch 更现代，支持 TrueColor 和更丰富的图标。

### 自定义

fastfetch 支持 JSON 配置文件，可精细控制每个模块的显示：

```bash
# 生成默认配置文件
fastfetch --gen-config

# 只显示指定模块
fastfetch --os --kernel --shell --terminal
```

### 配合欢迎消息

将 fastfetch 添加到 `~/.zshrc` 末尾即可每次打开终端自动显示：

```bash
# ~/.zshrc
fastfetch
```

## pfetch

[pfetch](https://github.com/dylanaraps/pfetch) 是同一作者（dylanaraps）用纯 Bash 编写的极简系统信息工具。它没有外部依赖，脚本体积不到 2KB，启动速度极快。

### 安装

```bash
# 直接下载脚本
curl -LO https://raw.githubusercontent.com/dylanaraps/pfetch/main/pfetch
chmod +x pfetch

# 移动到 PATH
mv pfetch /usr/local/bin/
```

### 使用

```bash
pfetch
```

输出风格非常简洁，没有 ASCII 艺术图，只用纯文字和颜色展示系统信息：

```
poloxue@macbook
--------------
OS        macOS 14.3
Kernel    Darwin 23.2.0
Uptime    3d 12h
Packages  342
Shell     zsh
```

### 优势

- **零依赖**——纯 Bash，任何系统都能跑
- **极快**——启动时间几乎可以忽略
- **代码可读**——整个脚本不到 100 行，可以当 Bash 学习素材
- **可定制**——直接修改脚本即可调整显示内容

## 三者对比

| 工具 | 语言 | 速度 | 依赖 | 维护状态 | 适合场景 |
|------|:----:|:----:|:----:|:--------:|----------|
| neofetch | Bash | 较慢 | 少 | 已停维 | 经典选择，要 ASCII 图 |
| fastfetch | Rust | 极快 | 无 | 活跃 | 日常推荐，功能最全 |
| pfetch | Bash | 极快 | 无 | 已停维 | 极简主义，追求速度 |

## 推荐方案

从实际使用角度，推荐以下组合：

```bash
# ~/.zshrc
# 先显示一条自定义欢迎消息
echo "🚀 欢迎回来，$(whoami)！美好的 $(date '+%Y-%m-%d') 开始了～"
echo ""

# 再用 fastfetch 显示系统信息
fastfetch
```

这样既有温度（自定义欢迎语），又有信息量（系统状态），而且 fastfetch 的启动速度足够快，不会让人等得不耐烦。
