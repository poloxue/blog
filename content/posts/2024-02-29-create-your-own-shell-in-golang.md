---
title: "我用 Go 开发了一个简易版 shell"
date: 2024-02-25T17:51:45+08:00
draft: true
comment: true
description: ""
---

看到 Github 有个 build-your-own-x 的仓库，我觉得挺有意思的，有不少有趣的实现。我就想着尝试实现下，多做些这样的小项目。

这个项目一方面可以帮助理解 shell 是什么？、命令解析和执行等概念。一个功能起算的shell，至少应该能解析用户输入的命令，并执行它们，同时还能提供一些内建命令的支持，比如改变当前目录或退出shell。

## 框架搭建

我从创建一个`Shell`结构体开始，这是整个 shell 程序的核心。`Shell`包含一个`bufio.Reader`来读取标准输入，和一个映射（map）来存储内建命令。

```go
type Shell struct {
	reader      *bufio.Reader
	builtinCmds map[string]BuiltinCmder
}
```

接着，我实现了`NewShell`构造函数，用于初始化`Shell`实例。这个函数返回一个新的`Shell`实例，其中包含了初始化的`bufio.Reader`。

为了方便扩展，我接下来添加了几个方法：`PrintPrompt`用于打印提示符，`ReadInput`用于读取用户输入，`ParseInput`用于解析输入并分割成命令名和参数，`ExecuteCmd`用于执行命令。

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
		if cmdName == "exit" {
			return nil
		}

		if err := s.ExecuteCmd(cmdName, cmdArgs); err != nil {
			fmt.Fprintln(os.Stderr, err)
			continue
		}
	}
}
```

## 实现退出功能

在初步测试中，我发现我的shell不支持退出能力。为了解决这个问题，我在`RunAndListen`循环中加入了对`exit`命令的检查。如果用户输入的是`exit`，循环将终止，从而退出shell。

## 支持改变目录

接下来，我注意到shell还不支持改变当前目录的能力。原因在于改变当前目录（即执行`cd`命令）需要对进程的工作目录进行修改，这种操作并不能通过创建新的进程来实现（像其他外部命令那样）。因此，我引入了内建命令的概念，并实现了`ChangeDirCommand`。

```go
type ChangeDirCommand struct{}

func (c *ChangeDirCommand) Execute(args ...string) error {
	if len(args) < 2 {
		return errors.New("Expected path argument")
	}
	err := os.Chdir(args[1])
	if err != nil {
		return nil
	}
	return nil
}
```

在 `Shell` 的 `ExecuteCmd` 方法中，我增加了对内建命令的检查和执行逻辑。如果命令是内建命令，则由对应的处理函数执行；否则，通过创建新的进程来执行外部命令。

## 输入支持引号

要支持单引号和双引号内的内容作为一个整体读入，而不是简单地以换行符（\n）判断命令的结尾，你需要对ReadInput方法进行修改，以便正确处理引号内的字符串。下面是一个简化的实现思路，它可以帮助你开始这个功能的开发。

你需要编写一个解析逻辑，以便在读取输入时能够识别和处理单引号'和双引号"。以下是一个实现的示例：

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

## 总结

通过开发这个简单的shell，我不仅加深了对Go语言的理解，还学到了许多关于操作系统如何处理命令和进程的知识。这个项目虽然简单，但它是一个非常实用的学习工具，帮助我将理论知识应用到实践中。未来，我计划继续扩展这个shell，添加更多内建命令和功能，使其更加完善。这个过程中，我期待学习到更多关于系统编程和Go语言的高级特性。
