---
title: "我的终端环境：Mac 和 Linux 下 7 个效率提升的命令（下）"
date: 2023-10-23T20:13:53+08:00
draft: true
comment: true
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
  - 正则搜索
    - rg -e '[0-9]{2}:[0-9]{2}'
  - 过滤文件
    - rg --exclude
  - 自动过滤
    - .gitgnore .ignore .rgignore
    - 隐藏文件，hidden file
    - rg -uuu hellworld
  - 搜索并替换
    - rg main ~/Code/golang-examples r main # 只替换输出，未修改文件
  - 文件过滤：
    - 通配符；
    - rg -glob 'code/golang/*.go' -i NewUser
  - rg -glob 'code/golang/**/*.go' -i NewUser
- 配置：
  - 配置文件修改 ripgrep 的默认行为；

[fzf](https://github.com/junegunn/fzf) - fuzzy finder 模糊匹配
- 说明：
- 安装：brew install fzf
- 用法：
  - fzf
  - vim `fzf`
- 别名:
  - alias cdg='cd_global() {cd $(fd --type directory $1 $2 | fzf)}; cd_global'

