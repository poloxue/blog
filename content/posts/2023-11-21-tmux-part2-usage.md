---
title: "2023 11 21 Tmux Part2 Usage"
date: 2024-03-20T17:26:15+08:00
draft: true
comment: true
description: "本文介绍 Tmux 的一些核心概念和基础使用。"
---


安装：
  - Homebrew 安装：
  ```zsh
  brew install tmux
  ```
  - 查看版本：
  ```zsh
  tmux -V
  ```

概念: 会话（session）-> 窗口（window） -> 窗格（pane）
使用：
- 说明：效果演示时，通过命令演示 Tmux 的操作效果，实际使用时依据命令作用绑定到快捷键。
- 会话：
  - 命令：
    - 新建会话：```tmux new|new-session -s <session-name>```
    - 脱离会话：```tmux detach|detach-session -s <session-name>```
    - 连接会话：```tmux attach|attach-session -s <session-name>```
    - 查看会话：```tmux list|list-sessions ```
    - 退出会话：```tmux kill|kill-session -s <session-name>```
    - 切换会话：```tmux switch|switch-session -t <session-name>```
    - 重命名：```tmux rename|rename-session -t <session-name> <new-session-name>```
  - 快捷键
    - 待补充；
- 窗格： 
  - 说明：如下是 pane 管理的常用命令。
  - 命令：
    - 划分窗格：
      - 垂直分屏：```tmux split-window``` 
      - 水平分屏：```tmux split-window -h```
    - 窗格切换：
      - 向上切换：```tmux select-pane -U```
      - 向下切换：```tmux select-pane -D```
      - 向左切换：```tmux select-pane -L```
      - 向右切换：```tmux select-pane -R```
    - 窗格交换：
      - 窗格上移：```tmux swap-pane -U```
      - 窗格下移：```tmux swap-pane -U```
      - 特定窗格：```tmux swap-pane -t|-s```
  - 快捷键
    - 待补充
- window 窗口
  - 命令：
    - 新建窗口：tmux new-window -n <window-name>
    - 切换窗口：
      - 指定编号：tmux select-window -t <window-number>
      - 指定名称：tmux select-window -t <window-name>
    - 重命名：tmux rename-window -t <window-name> <new-window-name>
- 其他
  - 查看快捷键：```tmux list-keys```
  - 查看命令列表：```tmux list-commands```
