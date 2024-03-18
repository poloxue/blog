---
title: "我用 Go 开发了一个简易版 shell"
date: 2024-03-01T17:51:45+08:00
draft: false
comment: true
description: "之前看到 Github 有个 build-your-own-x 的仓库，觉得挺有意思的，有不少有趣的实现。我就想着多尝试实现些这样的小项目，看看不同的领域。一方面提升我的编程能力，另外，也希望能发现一些不错的项目。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-29-create-your-own-shell-in-golang-01.png)

之前看到 Github 有个非常受欢迎的 build-your-own-x 仓库，觉得挺有意思的，有不少有趣的实现。我就想着多尝试实现些这样的小项目，看看不同的领域。一方面提升我的编程能力，另外，也希望能发现一些不错的项目。

今天的项目在 build-your-own-x 中也能找到，即 build your own shell。这个项目能帮助学习 Go 如何进行如 IO 输入输出、如何发起进程调用等操作。

## 核心流程

首先，我声明这是个简陋的 shell，但能帮助我们更好理解 Shell。它支持如提示符打印、读取用户输入、解析输入内容、执行命令，另外还支持开发内建命令。

如下是演示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-29-create-your-own-shell-in-golang-02.gif)

接下来，我将从零开始一步步复现我的整个开发过程。

## 框架搭建

我从创建一个 Shell 结构体开始，这是整个shell程序的核心，它其中包含一个 `bufio.Reader` 从标准输入读取用户输入。

```go
type Shell struct {
  reader      *bufio.Reader
}

func NewShell() *Shell {
  return &Shell{
    reader: bufio.NewReader(os.Stdin),
  }
}
```

如上，通过 `NewShell` 构造函数创建 `Shell` 实例。这个函数返回一个新的 Shell 实例，其中包含了初始化的 bufio.Reader。

为了方便扩展，接下来添加了几个方法，分别是：

- `PrintPrompt`用于打印提示符；
- `ReadInput`用于读取用户输入；
- `ParseInput`用于解析输入并分割成命令名和参数；
- `ExecuteCmd`用于执行命令。

定义如下：

```go
func (s *Shell) PrintPrompt()
func (s *Shell) ReadInput() (string, error)
func (s *Shell) ParseInput(input string) (string, []string)
func (s *Shell) ExecuteCmd(cmdName string, cmdArgs []string) error
```

它们就是核心流程中最重要的四个方法，都是在 `RunAndListen` 方法中被调用，如下所示：

```go
func (s *Shell) RunAndListen() error {
  for {
    s.PrintPrompt()

    input, err := s.ReadInput()
    if err != nil {
      fmt.Fprintln(os.Stderr, err)
      continue
    }

    cmdName, cmdArgs := s.ParseInput(input)

    if err := s.ExecuteCmd(cmdName, cmdArgs); err != nil {
      fmt.Fprintln(os.Stderr, err)
      continue
    }
  }
}
```

主函数 `main` 的代码不复杂，如下所示：

```go
func main() {
	s := NewShell()
	_ = s.RunAndListen()
}
```

通过 `NewShell` 创建 `Shell` 示例，调用 `RunAndListen` 监听用户输入即可。

接下来，我开始介绍其中每一步的实现过程。

## 打印提示符

首先，打印提示符的代码，非常简单，如下所示：

```go
func (s *Shell) PrintPrompt() {
  fmt.Print("$ ")
}
```

单纯的打印 `$` 作为提示符，更复杂的场景可以加上路径提示，如：

```
[~/demo/shell]$
```

修改后的代码如下所示：

```go
func (s *Shell) PrintPrompt() {
  // 获取当前工作目录
  cwd, err := os.Getwd()
  if err != nil {
    // 如果无法获取工作目录，打印错误并使用默认提示符
    fmt.Println("Error getting current directory:", err)
    fmt.Print("$ ")
    return
  }

  // 获取当前用户的HOME目录
  homeDir, err := os.UserHomeDir()
  if err != nil {
    fmt.Println("Error getting home directory:", err)
    fmt.Print("$ ")
    return
  }

  // 如果当前工作目录以HOME目录开头，则用'~'替换掉HOME目录部分
  if strings.HasPrefix(cwd, homeDir) {
    cwd = strings.Replace(cwd, homeDir, "~", 1)
  }

  // 打印包含当前工作目录的提示符
  fmt.Printf("[%s]$ ", cwd)
}
```

这是非常粗糙的拿到目录并打印出来。

通常 Shell 的提示符是可以自定义，有兴趣可以在这里扩展个接口类型，用于不同提示符的格式化实现。

## 读取用户输入

最简单的读取用户输入的代码，代码如下：

```go
func (s *Shell) ReadInput() (string, error) {
  input, err := s.reader.ReadString('\n')
  if err != nil {
    return "", err
  }

  return input, nil
}
```

按 `\n` 分割命令，分割出来的文本可以理解为一次执行请求。

但实际情况是在使用 Shell 时，我们会发现一些特殊符号是要处理，如引号。

例如：

```bash
[~/demo/shell]$ echo '
Hello World!
Nice to See you!
'
```

下面是一个简化的实现：

```go
func (s *Shell) ReadInput() (string, error) {
  var input []rune
  var inSingleQuote, inDoubleQuote bool

  for {
    r, _, err := s.reader.ReadRune()
    if err != nil {
      return "", err
    }

    // Check for quote toggle
    switch r {
    case '\'':
      inSingleQuote = !inSingleQuote
    case '"':
      inDoubleQuote = !inDoubleQuote
  }

    // Break on newline if not in quotes
    if r == '\n' && !inSingleQuote && !inDoubleQuote {
      break
    }

    input = append(input, r)
  }

  return string(input), nil
}
```

如上的代码中，逐一读取输入内容。程序中，通过判断当前是处于引号中，保证正确识别用户输入。

如果你读过我之前一篇文章，熟练使用 `bufio.Scanner` 类型，也可以用它提供的自定义分割规则的方式，在这个场景下也可以使用。我的完整源码 [goshell](https://github.com/poloxue/goshell) 就是基于 `Scanner` 实现的。

另外，这个输入不支持删除，如果我输出错了，只能退出重来，也是挺头疼的。如果要实现，要依赖于其他库实现。

## 解析输入

读取完成，通过 `ParseInput` 方法解析成 `cmdName` 和 `cmdArgs`，代码如下：

```go
func (s *Shell) ParseInput(input string) (string, []string) {
  input = strings.TrimSuffix(input, "\n")
  input = strings.TrimSuffix(input, "\r")

  args := strings.Split(input, " ")

  return args[0], args[1:]
}
```

真正的 Shell 肯定比这个强大的多了。最容易想到的，一次 shell 执行请求可能包含多个命令，甚至是 shell 脚本。

太复杂的能力实现起来太麻烦，我们可以支持一个最简单的能力，分号分割运行多个命令。

```bash
$ cd /; ls
```

我们修改代码，支持这个能力。

```go
type CmdRequest struct {
  Name string
  Args []string
}

func (s *Shell) ParseInput(input string) []*CmdRequest {
  subInputs := strings.Split(input, ";")

  cmdRequests := make([]*CmdRequest, 0, len(subInputs))
  for _, subInput := range subInputs {
    subInput = strings.Trim(subInput, " ")
    subInput = strings.TrimSuffix(subInput, "\n")
    subInput = strings.TrimSuffix(subInput, "\r")
    args := strings.Split(subInput, " ")
    cmdRequests = append(cmdRequests, &CmdRequest{Name: args[0], Args: args[1:]})
  }

  return cmdRequests
}
```

上面代码里，定义了一个新类型 `CmdRequest`，它用于保存从用户输入解析而来的命令名和命令参数。

由于修改了 `ParseInput` 的返回类型，`RunAndListen` 中的逻辑就要改动了。

如下所示：
```go
for {
  // ...

  cmdRequests := s.ParseInput(input)
  for _, cmdRequest := range cmdRequests {
    cmdName := cmdRequest.Name
    cmdArgs := cmdRequest.Args

    if err := s.ExecuteCmd(cmdName, cmdArgs); err != nil {
      fmt.Fprintln(os.Stderr, err)
      continue
    }
  }
}
```

到此，通过分号分割多命令也是支持的了。

## 命令执行

最后一步就是执行命令了。代码如下所示：

```go
func (s *Shell) ExecuteCmd(cmdName string, cmdArgs []string) error {
  cmd := exec.Command(cmdName, cmdArgs...)
  cmd.Stderr = os.Stderr
  cmd.Stdout = os.Stdout

  return cmd.Run()
}
```

我使用的是标准库 `exec` 包中的 `Command` 类型创建一个命令用于执行外部命令。

这个命令的标准输出和标准错误都被设置为当前进程的对应输出，这样命令的输出就可以直接显示给用户。

最后，通过调用 `cmd.Run()` 执行该命令即可。

## 退出功能

在初步测试中，我发现 shell 还不支持退出。为了解决这个问题，我在 `RunAndListen` 循环中加入了对 `exit` 命令的检查。

```go
for {
  cmdName := cmdRequest.Name
  cmdArgs := cmdRequest.Args

  if cmdName == "exit" {
    return nil
  }

  if err := s.ExecuteCmd(cmdName, cmdArgs); err != nil {
  }
}
```

如果用户输入的是`exit`，循环将终止，直接退出shell。

## 内建命令

现在，如果测试这个代码，看起来运转一切正常。但如果仔细测试，会发现它还不支持 `cd` 的能力。

为什么 `cd` 不能用？

因为改变当前目录要修改进程的工作目录，这种操作不能像其他外部命令那样通过创建新进程实现。因此，我引入了内建命令的实现，并实现第一个内建命令了`ChangeDirCommand`。

首先是搭建一个简单框架，定义一个接口：

```go
type BuiltinCmder interface {
	Execute(args ...string) error
}
```
任何实现了这个接口的类型都可以作为内建的命令。

在 `Shell` 类型新建了一个字段，名为 `builtinCmds `，修改定义如下：

```go
type Shell struct {
	reader      *bufio.Reader
	builtinCmds map[string]BuiltinCmder
}
```

并添加了一个方法，名为 `RegisterBuiltinCmd`：

```go
func (s *Shell) RegisterBuiltinCmd(cmdName string, cmd BuiltinCmder) {
	s.builtinCmds[cmdName] = cmd
}
```

在 `Shell` 的 `ExecuteCmd` 中新增了内建命令的执行：

```go

func (s *Shell) ExecuteCmd(cmdName string, cmdArgs []string) error {
  if cmd, ok := s.builtinCmds[cmdName]; ok {
    return cmd.Execute(cmdArgs...)
  }

  cmd := exec.Command(cmdName, cmdArgs...)
  // ...
}
```

现在，只要实现 `ChangeDirCommand`，并在 `main` 入口函数注册这个内建就行了。`ChangeDirCommand` 代码和入口注册代码，如下所示：

```go
type ChangeDirCommand struct{}

func (c *ChangeDirCommand) Execute(args ...string) error {
  if len(args) < 2 {
    return errors.New("Expected path argument")
  }
  return os.Chdir(args[1])
}

func main() {
	s := NewShell()

	s.RegisterBuiltinCmd("cd", &ChangeDirCommand{})

	_ = s.RunAndListen()
}
```

到此大功搞成，源码地址：[goshell](https://github.com/poloxue/goshell)

## 总结

通过开发这个简单的 shell，了解 Go 如何读取如用户输入，解析与执行用户命令。对 shell 的流程也有了一个大概了解。

未来，如果有想法，或许会继续扩展这个 shell，添加更多内建命令，可以将不同部分模块化，如 Prompt, Reader, Parser 和 Command 都是可以继续抽象以支持更多能力。

如果继续它的开发，期待学习到更多关于系统编程和 Go 语言的高级特性。而且shell 可不止这点能力，如果你了解 shell，使用过 bash 或是 zsh 等 shell 就知道它们是如何提高我们工作效率的了。

