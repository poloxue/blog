---
title: "我的终端环境：提升效率的 shell 命令（四）- 高效查找与搜索"
date: 2023-10-28T20:13:53+08:00
draft: true
comment: true
description: ""
---

[fd](https://github.com/sharkdp/fd)

- 说明：一款用于文件查找的命令，可用于替换默认命令 find；
- 安装：brew install fd
- 使用：
  - 简单搜索
    - fd pattern
  - 正则查询
    - fd '.*install'
    - fd '[0-9]{4}-[0-9]{2}'
  - 通配符
    - fd -g '*install*'
  - 无匹配符
    - fd -F '*install*'
  - 指定类型
    - fd -e md git
    - fd -e md golang
    - fd -e md zsh
  - 隐藏文件
    - fd -H config
  - 查询范围包含 gitignore
    - fd -I main.o
    - fd --no-ignore-vcs
  - 大小写敏感
    - fd -i readme.md # insensitive
    - fd -s readme.md # sensitive
  - 详细信息
    - fd -l .go
  - 大小过滤
    - fd -S +1000k
  - 查询目录
    - fd --type/-t directory golang
  - 查询并执行命令
    - fd --type/-t file -x wc -l
    - fd --type/-t file -X vim
- 性能：
  - 使用 hyperfine 测试，与 find 进行性能对比

[ripgrep](https://github.com/BurntSushi/ripgrep)
- 说明：用于文本搜索，功能与 grep 类似；
- 安装：brew install ripgrep
- 使用：
  - 匹配文本高亮显示；
  - 默认搜索当前目录
    - rg main
    - 递归搜索；
  - 搜索指定目录
    - rg main ~/Code/golang-examples
  - 搜索指定文件
    - rg main ~/Code/golang-examples/main.go
  - 禁用递归搜索
    - cd ~/Code/golang-examples
    - rg main -g '!/*/*'
  - 正则搜索
    - rg -e '[0-9]{2}:[0-9]{2}'
  - 过滤文件
    - 排除文件：rg -g '!directory'
  - 自动过滤
    - 忽略文件
      - 默认过滤，包括 .gitgnore .ignore .rgignore 中的文件；
      - 禁用该功能 --no-ignore
    - 隐藏文件，hidden file
    - 禁用所有 rg -uuu hellworld
  - 文件过滤：
    - 通配符；
    - 包含：rg -g 'pattern' NewUser
    - 排除：rg -g !pattern' 
  - 大小写敏感：
    - 默认大小写敏感：rg pattern
    - 大小写不敏感：rg -i pattern
    - smartcase：rg -S pattern
  - 搜索并替换
    - rg main ~/Code/golang-examples r main # 只替换输出，未修改文件
- 配置：
  - 配置文件修改 ripgrep 的默认行为；

[fzf](https://github.com/junegunn/fzf) 
- 说明：fuzzy finder，通用的命令行模糊查找器；
- 安装：brew install fzf
- 用法：
  - Home 目录
    - fzf
  - 与其他命令结合
    - 将其他命令的输出作为 fzf 的搜索内容；
      - echo "one\ntwo\nthree\nfour" | fzf
      - ls | fzf
    - 将搜索结果作为其他命令的输入；
      - vim `fzf`
  - 命令选项查询字符串
    - fzf --query "pattern"
    ode"
- 别名:
  - alias cdg='cd_global() {cd $(fd --type directory $1 $2 | fzf)}; cd_global'

