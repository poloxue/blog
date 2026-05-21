---
title: "自定义提示主题"
weight: 5
---

# 自定义提示主题

不知道你是否想过自定义 Shell 提示符主题能带来的不仅是终端美观度的提升，还能通过视觉优化增强了工作效率呢？

在众多 shell 提示符主题中，Powerlevel10k 因为支持高度可定制和丰富的功能选，非常值得推荐。本文基于这个主题介绍 zsh 主题插件 powerlevel10k，包括它的安装和配置自定义。

## 什么是 powerlevel10k?

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-16.png)

Powerlevel10 是一款 zsh 的主题，强调性能、灵活性和开箱即用，但同时自定义能力极强。前面介绍 zsh 轻量级框架 oh-my-zsh 时，提到过一些 zsh 主题，而通过 p10k（powerlevel10k 的简称）的自定义配置化能力，同样能配置出覆盖出之前主题的类似效果，当然相对而言，也更加强大。

效果展示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-13.png)

## 安装依赖字体

在安装 powerlevel10k 前，要先安装它依赖的字体：[NerdFont](https://github.com/ryanoasis/nerd-fonts#font-installation)。不同系统下的安装方法，查看[它的文档](https://github.com/ryanoasis/nerd-fonts#font-installation)。

简单说下 Nerd Fonts 字体。它是一系列开源字体的集合，被特别增强，它包含大量的图标和符号，如开发工具、编程语言和版本控制系统的图标。这些字体对于提高我们终端和编辑器的视觉体验和功能性有着极大帮助。

有了它，我们的终端才能显示一些复杂字体甚至是图标。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-14.png)

MacOS 的话，可直接通过 Homebrew 快速安装：

```bash
brew tap homebrew/cask-fonts
brew install font-hack-nerd-font
```

安装完成，配置终端字体，进入 iTerm2 Settings -> Profiles -> Text -> Font -> MesloLGS NF 即可。

现在，我们终端就支持 NerdFont 字体了。

如何测试？

接下来安装 Powerlevel10k 时，它会提示我们检查字体是否正确安装。

## 安装 powerlevel10k

先通过如下的命令下载插件源码放到指定的位置。

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

在 `~/.zshrc` 配置启动 `powerlevel10k` 主题插件。

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

现在我们只要重新终端或执行 `source ~/.zshrc` 生效配置就会进入到配置向导流程中了。

## 配置 powerlevel10k

终端将进入到 powerlevel10k 的配置向导后，我们通过回答问题来完成自定义的安装流程。这个流程有点繁琐，我将一步一步介绍，要跟着操作可能更有感觉吧。

### 前置问题

首先，为了确认 NerdFont 字体安装成功，正式开始配置前，要先提出几个问题，识别图片是否如描述说的那样。我们回答后才能进入到主题的自定义。


这些问题有诸如：

- Does this look like a diamond (rotated square)? 这是看起来钻石吗？
- Does this look like a lock? 这看起来是锁吗？
- Does this look like an upwards arrow? 这看起来是向上箭头吗？

等等。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-17.png)
![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-18.png)
![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-19.png)

如果已经成功安装配置了字体，这些问题看起来就像逗傻子一样，按实际情况回答问题即可。如果字体安装成功，都是选择 "Yes" 即可。

### 开始配置

进入到正式阶段，按步骤配置自己的提示符风格，下面一共 12 步，都是非常简单的回答问题。考虑到第一次配置，有懵逼的可能性，我把全部的步骤都列出来了。

选择提示符风格，分别是 Lean、Classic、Rainbow 和 Pure。我选风格 3，大众所爱的风格，彩虹 Rainbow，则输入 3。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-01.png)

 字符集设置，毫无疑问，配置 unicode，选择 1。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-02.png)

提示符显示当前时间风格。我选择 1，只显示命令耗时，不显示当前时间。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-03.png)

提示符分隔符，也就是 src/master 之间的符号风格。我钟爱箭头，选择 1 -> Angled.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-04.png)

提示符头部风格，就是 `master >` 的风格选择。毫无疑问，我钟爱箭头，选择 1 - Sharp。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-05.png)

提示符尾部风格，钟爱箭头，但不是双箭头，选择 1 - Flat。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-06.png)

提示符高度，显示一行还是两行，体验过两行，还是一行更紧凑一些，选择 1 - one line。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-07.png)

两个命令键的间距，我喜欢两行离的近一点，选择 1 - Compact。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-08.png)

提示符 Icons，多点 Icon 更帅，否则看起来就和一般主题没区别了，so 选择 2 - Many Icons。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-09.png)

提示符丰富度，增加一些文本描述，帮助理解提示符中字符含义。还是简洁为美，毕竟空间不能占用太多，而且含义简单，无需文本辅助。我选择 1 - Consice。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-10.png)

这是什么配置ne？提示符瞬闪？好像命令执行后提示符就立刻消失，只保留在最新的提示符，先选择 n - No 看看效果吧。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-11.png)

提示符高性能模式，是否启用。推荐启用，就启用吧 1-verbose，如果发现有兼容问题，在重新配置 off。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-12.png)

到此基本全部的配置都已经完成，powerlevel10k 命令行提示符的最终效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-20-zsh-theme-powerlevel10k-13.png)

## 重新配置

在配置完成后，如果希望重新配置，重启整个流程，直接执行 `p10k configure` ，它会重新打开配置向导。

## 配置文件

通过 powerlevel10k 的配置导航能快速自定义提示符的主题风格，但如想更细粒度的配置，可直接在 `$HOME/.p10k.zsh` 配置，配置导航只是最粗粒度的配置方式。

如配置提示符两侧内容，通过 `~/.p10k.sh` 中的变量 `POWERLEVEL9K_LEFT_PROMPT_ELEMENTS` 和 `POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS` 设置。

配置文件的内容，如下所示：

```zsh
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
  # 操作系统图标
  os_icon       
  # 当前目录
  dir           
  # 版本控制信息
  vcs           
  # 提示符
  # prompt_char 
)

typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
  # 最后一条命令的退出码 
  status                  
  # 最后一条命令的执行时间
  command_execution_time  
  ...
  # 当前时间
  # time
  ...
)
```

前面配置时，设置了不显示当前时间，可以通过打开 `POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS` 中的 `time` 重新显示时间。

还有前面对提示符瞬闪的配置，transient 设置为 off。如果希望启用，同时有不重新经历一次配置向导，直接进入 `~/.p10k.zsh` 配置变量 `POWERLEVEL9K_TRANSIENT_PROMPT=always`，即可。

```zsh
# Transient prompt works similarly to the builtin transient_rprompt option. It trims down prompt
# when accepting a command line. Supported values:
#
#   - off:      Don't change prompt when accepting a command line.
#   - always:   Trim down prompt when accepting a command line.
#   - same-dir: Trim down prompt when accepting a command line unless this is the first command
#               typed after changing current working directory.
typeset -g POWERLEVEL9K_TRANSIENT_PROMPT=always
```

Powerlevel10k 还进一步集成了对各种工具的支持，包括但不限于 npm、k8s、Python 和 Go。在 `~/.p10k.zsh` 中配置相应的提示元素，如 `node_version`、`kubecontext`、`python_version` 和 `go_version`。

这些都是位于 `POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS` 配置项下。

```
typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
  # ...
  virtualenv 
  anaconda 
  pyenv
  goenv
  nodenv
  nvm
  nodeenv
  node_version
  go_version
  rust_version 
  dotnet_version
  php_version            
  laravel_version        
  java_version           
  package                
  rbenv                 
  rvm                   
  fvm                   
  luaenv                
  jenv                  
  plenv                 
  perlbrew              
  phpenv               
  scalaenv 
  haskell_stack   
  kubecontext      
  terraform         
  terraform_version   
  aws                
  aws_eb_env           
  azure                 
  gcloud                 
  google_app_cred         
  toolbox    
  context     
  nordvpn      
  ranger        
  nnn            
  lf              
  xplr             
  vim_shell        
  midnight_commander
  nix_shell         
  chezmoi_shell      
  vpn_ip            
  load                 
  disk_usage           
  ram                  
  swap                  
  todo                    
  timewarrior             
  taskwarrior             
  per_directory_history   
  cpu_arch              
  time                  
  newline
  ip                    
  public_ip            
  proxy                 
  battery               
  wifi                 
  example
)
```

这些配置让我们也可以在提示符中直观地看到当前环境的版本信息，以及 Kubernetes 上下文等关键信息，从而使我们的工作流程更加高效、直观。

这里只是简单介绍了 powerlevel10k 的配置。想了解它的更多能力，可以在 `~/.p10k.sh` 继续探索。

## 总结

本文介绍了 powerlevel10k 的安装与配置。如果希望你命令行提示能给你提供更多信息，减少使用终端时的一些心智负担，如少执行几次版本，分支或所在上下文查询的命令，强烈推荐安装 powerlevel10k！


