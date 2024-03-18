---
title: "我用 Python 为 iTerm2 开发一个类似 tmuxifier 的工具"
date: 2024-02-25T08:00:00+08:00
draft: false
comment: true
description: "我在思考如何提高终端工作效率时，想到了在 iTerm2 中实现一个类似于 tmuxifier 布局管理工具。如果你不了解 tmuxifier，简单来说，它是 tmux 的布局管理工具。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-26-build-an-itermifier-01.png)

我在思考如何提高终端工作效率时，想到了是否能给予 iTerm2 实现一个类似于 tmuxifier 布局管理工具。如果你不了解 tmuxifier，简单来说，它是 tmux 的布局管理工具。

它的作用如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-26-build-an-itermifier-02-v1.gif)

想到不如做到！

这篇文章记录了我从零开始，一步步构建这个工具的过程。

## 设计阶段

正式开发这个工具前，先调研下可行性还是很重要的。

### iTerm2 支持 Python API

首先，我了解到 iTerm2 支持通过 Python 脚本控制终端，这为我开发这样一个工具提供了可能性。

它提供大量的 API，如更换背景、创建窗格，切换 Tab 等都是支持的。具体可查看文档：[Python-API](https://iterm2.com/python-api/)。

### 设置开发环境

为了顺利开始，首先确保我的 iTerm2 （3.4.23）支持 Python API，还有我的机器上配置了 Python 环境。

接着，我安装了 iTerm2 的 Python API 所需的库。

```bash
$ pip install iterm2
```

### 配置文件的设计

我选择用 YAML 作为配置文件的格式，因为它的可读性好，格式简洁，有现成的解析库。我要做的就是通过 YAML 文件要定义布局，包括窗口、窗格和命令。

这是 tmuxifier 的配置文件。

```bash
# 设置窗口名称和根目录
session_root "~/Projects/my-project"
window_name "my-project"

# 创建一个新窗口，并分割成两个窗格
new_window "my-project"
split_h 50

# 第一个窗格运行 vim
run_cmd "vim" 0

# 第二个窗格运行 go run main.go
run_cmd "go run main.go" 1

# 设置主窗格
select_pane 0
```

我将其映射成 YAML 配置文件，如下所示：

```yaml
layout:
  window_root: "~/Projects/my-project"
  window_name: "my-project"
  panes:
    - percentage: 50
      commands:
        - "vim"
    - type: "split_h"
      commands:
        - "git status"
  main_pane: 0
```

这个配置旨在创建一个布局，其中包含两个窗格，一个运行 `vim`，另一个显示 `go run main.go`，而鼠标聚焦在 `vim` 窗格下。

配置 `yaml` 文件的解析，我使用 Python 的 `yaml` 库来解析配置文件，

```python
import yaml

# 解析 YAML 配置
with open('layout.yaml', 'r') as f:
    config = yaml.safe_load(f)
layout = config["layout"]
```

这样我就能得到一个 `layout` 字典，其中包含了所有的布局信息。

接下来，主要就是将这个配置转换为 iTerm2 的操作，也就是根据配置指示，创建窗口和窗格，以及在窗格中执行特定命令。

## 实现布局管理工具

首先，创建一个 Python 脚本，假设就叫 itermifier 吧。这个脚本根据我的 YAML 配置创建窗格并执行命令。

### 准备部分

在开始构建我的 iTerm2 布局管理工具前，还需要准备一些基本的变量，这些变量将在创建布局的过程中被频繁使用。

首先，基于 iTerm2 API 创建主程序入口和获取 iTerm2 应用实例，如下：

```python
async def create_layout(connection):
  app = iterm2.async_get_app(connection)

iterm2.run_until_complete(create_layout)
```

代码中，定义了一个名为 `create_layout` 的异步函数，它是整个工具的核心。`connection` 参数是与 iTerm2 应用程序的连接，它是执行所有 iTerm2 API 调用的必要条件。iTerm2 的应用实例 `app` 就是从它获取而来的。

这是后续所有操作的起点。

### 窗口和标签页

接着，获取到当前活跃的窗口和标签页：

```python
window = app.current_terminal_window
tab = window.current_tab
```

这些步骤确保了我能在正确的上下文中操作 iTerm2，无论是分割窗格还是运行命令。

接下是创建了一个名为 `sessions` 的列表，用于存储标签页中的会话。

```python
# 预先创建的会话列表
sessions = [tab.current_session] 
```

这个列表用于存储当前这个标签页的所有会话，初始内容中的是当前标签页会话，通过 `tab.current_session` 获取。

### 分割窗格

接下来，就是按配置分割窗格。通过 `for` 循环窗格配置。

```python
for index, pane in enumerate(layout["panes"]):
  pass
```

以下是分割窗格的实现代码：

```python
vertical = False if pane["type"] == "split_h" else True
session = await tab.current_session.async_split_pane(vertical=vertical)

session = sessions.append(session)
```

对于每个窗格配置，我调用 iTerm2 API 的 `async_split_pane` 方法，根据需要进行水平或垂直分割。

### 执行预设命令

每个窗格创建后，通过 `async_send_text` 方法执行各个 pane 的预设命令。

```python
for cmd in pane.get("commands", []):
      await session.async_send_text(f"{cmd}\n")  # pyright: ignore
```

到这里，基本上已经实现了我想要的所有能力了。

### 回到主窗格

最后，我实现了将焦点移回主窗格的功能。

前面通过维护一个 session 列表，保存了所有创建的窗格会话，这样，我可以在所有操作完成后，按 `main_pane` 获取最终聚焦的窗口。

```python
main_pane_index = layout["main_pane"]
await sessions[main_pane_index].async_activate()
```

到此，基本全部完成了。

效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-26-build-an-itermifier-03.gif)

这里面，我省略一些代码，完成代码查看这个地址：[itermifier](https://gist.github.com/poloxue/c2b1a4c839750f1c5c0cba65c74a223b)。

### 结语

通过这个过程，我成功实现了一个在 iTerm2 中自动管理布局的工具。虽然过程中遇到了一些挑战，但最终能够通过编程方式定义和启动我的工作环境，还是很爽的。

要说遗憾的话，就是不支持大小比例的调整，可惜了。
