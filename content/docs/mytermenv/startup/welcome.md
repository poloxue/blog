---
date: 2024-04-07
title: "欢迎消息配置"
weight: 1
---

# 启动消息

每次打开终端，你看到的第一样东西是什么？默认情况下可能是 "Last login: Tue Mar 12 ..." 加上一行空的提示符。其实这个空间完全可以利用起来——显示系统状态、项目信息、或者纯粹让你心情好的东西。

## motd 与欢迎脚本

macOS/Linux 系统的 `/etc/motd`（Message of the Day）文件可以设置登录后显示的欢迎信息。

### 系统级 motd

```bash
# 编辑系统级欢迎消息
sudo vim /etc/motd

# 示例
cat /etc/motd
```

输出：

```
╔══════════════════════════════╗
║   Welcome to the Terminal   ║
║   Keep calm and code on     ║
╚══════════════════════════════╝
```

### 用户级欢迎消息（推荐）

我更推荐在 `~/.zshrc` 中定义自己的欢迎消息，这样更灵活：

```bash
# ~/.zshrc 末尾
echo "🚀 Hello, $(whoami)! Today is $(date '+%A, %Y-%m-%d')"
```

## 显示待办与提醒

结合 todo.txt 或提醒工具，在终端启动时显示当天待办：

```bash
# ~/.zshrc
echo ""
echo "📋 今日待办："
if [ -f ~/todo.txt ]; then
  grep "$(date '+%Y-%m-%d')" ~/todo.txt || echo "   ✅ 今天没有待办"
fi
echo ""
```

## 显示系统负载

更实用的做法是显示系统的关键状态：

```bash
function show_sys_status() {
  local load=$(uptime | awk -F'load averages:' '{print $2}')
  local disk=$(df -h / | awk 'NR==2 {print $4, "free on", $NF}')
  local mem=$(vm_stat | awk '/free/ {print $3}' | tr -d '.')
  local mem_mb=$((mem * 4096 / 1024 / 1024))

  echo "⚡ 系统状态"
  echo "   CPU负载:${load}"
  echo "  磁盘剩余: ${disk}"
  echo "  空闲内存: ${mem_mb}MB"
}
```

## 一个完整的欢迎脚本示例

```bash
# ~/.zshrc 或在 ~/.oh-my-zsh/custom/ 下创建一个新文件
function welcome_message() {
  clear
  echo "╔══════════════════════════════════════╗"
  echo "║   $(date '+%Y-%m-%d %A')"
  echo "║   $(whoami)@$(hostname -s)"
  echo "╠══════════════════════════════════════╣"
  echo "║   📂 $(pwd)"
  echo "║   ⏱ Uptime: $(uptime | awk -F'up' '{print $2}' | awk -F',' '{print $1}')"
  echo "╚══════════════════════════════════════╝"
  echo ""
}

# 调用
welcome_message
```

如果你想每次打开终端都显示，直接调用 `welcome_message`；如果只想在需要时使用，把它定义成函数，需要时手动执行。

## 小结

终端启动消息是一个被很多人忽略的细节。花 5 分钟配置一个简单的欢迎面板，每次打开终端都能获得一点好心情——以及实用的系统状态信息。别让终端的第一眼永远是冰冷的 "Last login:"。
