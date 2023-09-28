---
title: "iTerm2 启动时进入 Tmux 模式"
date: 2023-09-15T15:06:02+08:00
draft: false
comment: true
tags: ["tmux", "iTerm2"]
---

介绍一个最快速的方式使 iTerm2 启动默认进入 Tmux 模式。默认情况下，每次启动 iTerm2，还需要一步输入 tmux attach 进入到 tmux 模式下。

我用 Tmux 是为了管理不同项目的工作区，常见的 IDE 一般够提供了打开提供给用户一个选择项目的界面。自然而然，iTerm2 + Tmux 是否也能实现类似的能力呢？

非常简单！

**核心脚本介绍**

首先，一段 bash 脚本:

```bash
tmux ls && read -p "Select a session<default>:" tmux_session && tmux attach -t ${tmux_session:-default} || tmux new -s ${tmux_session:-default}
```

这段脚本的说明如下：

- `tmux ls`, 先输出当前可用的 session 列表，供用户输入使用；
- `read xxx`, 读取用户输入，将希望打开的会话名称存入 tmux_session 中；
- `tmux attach`，尝试打开会话，如果 tmux_session 为空，打开 default 会话；
- `tmux new`，如果开启失败，尝试创建一个新的会话；

说明：由于 `read` 命令使用了`-p`，必须要使用 bash 运行这段脚本。

**配置 iTerm2 启动加载**

选择 Preference -> Profile -> General -> Command -> Select "Login Sell"，并将代码放到 "Send text at start" 的输入框即可。

效果如下：

```bash
hello: 1 windows (created Tue Sep 12 17:13:54 2023) (group hello) (attached)
default: 1 windows (created Wed Sep 13 19:54:36 2023) (group default)
Select a session<default>:
```

输入 "hello"，即可进入 "hello" 会话。简洁高效，有点想 IDE 启动的之后，让我们选择项目的感觉。

如果希望会话列表足够简洁，只有会话名称，可用 `tmux ls` 的输出格式化默认，将 `tmux ls` 进化为 `tmux -F '\#{session_name}'`；

```bash
hello
default
```

**尝试升级改进**

能不能只输入索引编号，实现快速选择呢？这种简单的 shell script 不容易实现。写了一段 Python 脚本，如下所示：

```python
#!/usr/bin/env python
import os

output = os.popen("tmux ls -F \\#{session_name}").read()
sessions = output.strip().split("\n")

print("Sessions:")
for index in range(len(sessions)):
    print(f"{index} - {sessions[index]}")

input_value = input("Please select a session <Index or Name>(default):")
if input_value.isdigit():
    sess_name = sessions[int(input_value)]
elif not input_value:
    sess_name = "default"
else:
    sess_name = input_value

if sess_name not in sessions:
    answer = input(f"New a session `{sess_name}`?(Y/N)")
    answer == "Y" and os.system(f"tmux new -s {sess_name}")  # pyright: ignore
else:
    os.system(f"tmux attach -t {sess_name}")
```

按上面的方式将脚本配置到 iTerm2 中，启动效果:

```bash
❯ ~/tmux_selector
Sessions:
0 - hello
1 - default
Please select a session <Index or Name>(default):
```


这里只是一个小 demo，tmux ls 的输入还有其他格式化变量，有兴趣可以继续扩展。代码片段 [github 地址](https://gist.github.com/poloxue/37d3d79b35964ab8d885296b84ab4b5a)。

总体上的体验还不错，但如果使用自带的功能，岂不是更好？继续探索吧。

**使用 tmux 内置菜单树**

进入 tmux 模式后，默认可以用快捷键 prefix-key + s 启动 tmux 选择菜单：

```zsh
hello: 1 windows (created Tue Sep 12 17:13:54 2023) (group hello) (attached)
default: 1 windows (created Wed Sep 13 19:54:36 2023) (group default)
```

而且可以使用 jk 移动选择菜单。这个有没有办法在 iTerm2 启动时直接启动呢？想法很好，实现起来也不难。

首先，通过 `tmux list-keys` 检查 prefix-key + s 绑定的命令 tmux 命令。

```zsh
$ tmux list-keys
...
bind-key    -T prefix       s                    choose-tree -Zs
...
```

通过 `choose-tree -Zs` 即可拉起菜单。在 tmux 模式下，使用 `tmux choose-tree -Zs` 测试一下效果，选择窗口成功拉起。

那么，接下来的问题是，如何配置到 iTerm2 启动命令呢？

因为这个命令必须在 tmux 模式下运行。如何做？通过 `tmux attach\; <other commands running on tmux mode>`  即可实现。

完整命令如下：

```zsh
$ tmux attach\; choose-tree -zS
```

试着在非 tmux 模式下测试下这段代码，看看是否能成功拉起菜单。确认没有问题后，将上面的语句配置到你的 iTerm2 启动时执行即可。

最后，为了防止首次进入没有 session，进一步优化，创建默认的 default 会话窗口。

完整 shell 代码如下：

```bash
$ tmux attach\; choose-tree -Zs || read -p "Create a default session?(Y/N)" anwser && [[ "${anwser}" == "Y" ]] && tmux new -t default
```

但这里还有个缺点，默认 q 只能退出菜单，终端还是在 tmux 模式会话中，还要多一步 detach 才能退出，这算是和我们一般认为的步骤不太一样的地址吧。

