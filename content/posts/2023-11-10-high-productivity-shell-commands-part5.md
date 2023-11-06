---
title: "我的终端环境：高效 shell 命令（四）- 现代 Unix 命令集"
date: 2023-11-08T15:40:41+08:00
draft: true
comment: true
description: "本文介绍一个更符合现代风格的 Unix 命令的集合"
---

类 Unix 系统发展多年，不少老古董命令依然占据着我们终端的绝大部分时间，，大有经久不衰、经典永留存的趋势。但不得不提，经过这么多年，这些老古董们在体验上还是那么差强人意。

从本文开始，我将用四篇文章介绍一系列可用于提升终端操作效率的 Unix 命令，这些命令更具现代风格，希望能让你眼前一亮。

## 命令集合

推荐一个 github 仓库：[modern-unix](https://github.com/ibraheemdev/modern-unix)，其中收录了大量的更具现代风格的命令，可用于替换一大波一直在用的老古董命令。

举几个命令，如最常用的 ls、cd、grep、find 等等命令，其中都有合适的替代命令。

## 一键安装

```zsh
brew install bat exa lsd git-delta dust duf broot fd ripgrep ag fzf mcfly choose jq sd cheat tldr bottom glances gtop hyperfine gping procs httpie curlie xh zoxide dog entr lf
```

## 快速一览

让我们快速一览这个 Unix 命令集合，其中的大部分命令是来自于 [modern-unix](https://github.com/ibraheemdev/modern-unix) 这个仓库。

1. [lsd](https://github.com/lsd-rs/lsd) - 号称 "下一代 ls 命令"，与 ls 兼容。

- lsd --long --git

2. [delta](https://github.com/dandavison/delta) - 用于支持 git、diff 和 grep 的语法高亮分页器；

- diff -u main1.go main2.go | delta

3. [dust](https://github.com/bootandy/dust) - 

4. [duf](https://github.com/muesli/duf) -

5. [broot](https://github.com/Canop/broot) - 

6. [ag](https://github.com/ggreer/the_silver_searcher) - 

7. [fzf](https://github.com/junegunn/fzf) - 

8. [mcfly](https://github.com/cantino/mcfly) - 

9. [choose](https://github.com/theryangeary/choose) - 

14. [jq](https://github.com/jqlang/jq) - 

15. [sd](https://github.com/chmln/sd) - 

16. [cheat](https://github.com/cheat/cheat) - 

17. [tldr](https://github.com/tldr-pages/tldr) -

18. [bottom](https://github.com/ClementTsang/bottom) - 

19. [glances](https://github.com/nicolargo/glances) - 

20. [gtop](https://github.com/aksakalli/gtop) - 

21. [hyerfine](https://github.com/sharkdp/hyperfine) - 

22. [gping](https://github.com/orf/gping) - 

23. [procs](https://github.com/dalance/procs) - 

24. [httpie](https://github.com/httpie/cli) - 

25. [curlie](https://github.com/rs/curlie) - 

26. [xh](https://github.com/ducaale/xh) - 

27. [zoxide](https://github.com/ajeetdsouza/zoxide) - 

28. [dog](https://github.com/ogham/dog) - 

29. [entr](https://github.com/eradman/entr)

30. [lf](https://github.com/gokcehan/lf) -
