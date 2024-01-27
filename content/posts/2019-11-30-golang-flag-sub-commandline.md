---
title: "Go 命令行解析 flag 包之通过子命令实现看 go 命令源码"
date: 2019-11-30T15:33:32+08:00
draft: false
comment: true
tags: ["Golang"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-30-golang-flag-sub-commandline-01.png)

[上篇文章](https://mp.weixin.qq.com/s/rzgYifoMzWOO_PD0-2UIpw) 介绍了 flag 中如何扩展一个新的类型支持。本篇介绍如何使用 `flag` 实现子命令，总的来说，这篇才是这个系列的核心，前两篇只是铺垫。

前两篇文章链接如下：

[Go 命令行解析 flag 包之快速上手](https://mp.weixin.qq.com/s/rzgYifoMzWOO_PD0-2UIpw)  
[Go 命令行解析 flag 包之扩展新类型](https://mp.weixin.qq.com/s/rzgYifoMzWOO_PD0-2UIpw)

希望看完本篇文章，如果再阅读 go 命令的实现源码，至少在整体结构上不会迷失方向了。

# FlagSet

正式介绍子命令的实现之前，先了解下 flag 包中的一个类型，`FlagSet`，它表示了一个命令。

从命令的组成要素上看，一个命令由命令名、选项 `Flag` 与参数三部分组成。类似如下：

```bash
$ cmd --flag1 --flag2 -f=flag3 arg1 arg2 arg3
```

`FlagSet` 的定义也正符合了这一点，如下：

```go
type FlagSet struct {
	// 打印命令的帮助信息
	Usage func()

	// 命令名称
	name          string
	parsed        bool
	// 实际传入的 Flag
	actual        map[string]*Flag
	// 会被使用的 Flag，通过 Flag.Var() 加入到了 formal 中
	formal        map[string]*Flag

	// 参数，Parse 解析命令行传入的 []string，
	// 第一个不满足 Flag 规则的（如不是 - 或 -- 开头），
	// 从这个位置开始，后面都是
	args          []string // arguments after flags
	// 发生错误时的处理方式，有三个选项，分别是
	// ContinueOnError  继续
	// ExitOnError      退出
	// PanicOnError     panic
	errorHandling ErrorHandling
	output        io.Writer // nil means stderr; use out() accessor
}
```

包含字段有命令名 `name`，选项 Flag 有 `formal` 和 `actual`，参数 `args`。

如果有人说，`FlagSet` 是命令行实现的核心，还是比较认同的。之所以前面一直没有提到它，主要是 flag 包为了简化命令行的处理流程，在 `FlagSet` 上做了进一步的封装，简单的使用可以直接无视它的存在。

flag 中定义了一个全局的 `FlagSet` 类型变量，`CommandLine`，用它表示整个命令行。可以说，`CommandLine `是 `FlagSet` 的一个特例，它的使用模式较为固定，所以在它之上能提供了一套默认的函数。

前面已经用过的一些，比如下面这些函数。

```go
func BoolVar(p *bool, name string, value bool, usage string) {
	CommandLine.Var(newBoolValue(value, p), name, usage)
}

func Bool(name string, value bool, usage string) *bool {
	return CommandLine.Bool(name, value, usage)
}

func Parse() {
	// Ignore errors; CommandLine is set for ExitOnError.
	CommandLine.Parse(os.Args[1:])
}
```

更多的，这里不一一列举了。

接下来，我们来脱掉这层外衣，梳理下命令行的整个处理流程吧。

# 流程解读

`CommandLine` 的整个使用流程主要由三部分组成，分别是获取命令名称、定义命令中的实际选项和解析选项。

命令名称在 `CommandLine` 创建的时候就已经指定了，如下：

```go
CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```

名称由 `os.Args[0]` 指定，即命令行的第一个参数。除了命令名称，同时指定的还有出错时的处理方式，`ExitOnError`。

接着是定义命令中实际会用到的 `Flag`。

核心的代码是 `FlagSet.Var()`，如下所示：

```go
func (f *FlagSet) Var(value Value, name string, usage string) {
	// Remember the default value as a string; it won't change.
	flag := &Flag{name, usage, value, value.String()}

	// ...
	// 省略部分代码
	// ...

	if f.formal == nil {
		f.formal = make(map[string]*Flag)
	}
	f.formal[name] = flag
}
```

之前使用过的 `flag.BoolVar` 和 `flag.Bool` 都是通过 `CommandLine.Var()`，即 `FlagSet.Var()`， 将 `Flag` 保存到 `FlagSet.formal` 中，以便于之后在解析的时候能将值成功设置到定义的变量中。

最后一步是从命令行中解析出选项 `Flag`。由于 `CommandLine` 表示的是整个命令行，所以它的选项和参数一定是从 `os.Args[1:]` 中解析。

`flag.Parse` 的代码如下：

```go
func Parse() {
	// Ignore errors; CommandLine is set for ExitOnError.
	CommandLine.Parse(os.Args[1:])
}
```

现在的重点是要了解 flag 中选项和参数的解析规则，如 `gvg -v list`，按什么规则确定 `-v` 是一个 Flag，而 `list` 是参数的呢？

如果继续向下追 `Parse` 的源码，在 `FlagSet.parseOne` 中将发现 `Flag` 的解析规则。

```go
func (f *FlagSet) ParseOne()
	if len(f.args) == 0 {
		return false, nil
	}
	s := f.args[0]
	if len(s) < 2 || s[0] != '-' {
		return false, nil
	}
	numMinuses := 1
	if s[1] == '-' {
		numMinuses++
		if len(s) == 2 { // "--" terminates the flags
			f.args = f.args[1:]
			return false, nil
		}
	}
	// ...
}
```

三种情况下会终止解析 `Flag`，分别是当命令行参数全部解析结束，即 `len(f.args) == 0`，或长度小于 2，但第一位字符不是 `-`，或者参数长度等于 2，且第二个字符是 `-`。之后的内容会继续当作命令行参数处理。

如果没有子命令，命令的解析工作到此就基本完成了，再往后就是业务代码的开发了。那如果 `CommandLine` 还有子命令呢？

# 子命令

子命令和 `CommandLine` 无论是形式还是逻辑上，基本没什么差异。形式上，子命令同样包含选项和参数，逻辑上，子命令的选项和参数的解析规则与 `CommandLine` 相同。

一个包含子命令的命令行，形式如下：

```bash
$ cmd --flag1 --flag2 subcmd --subflag1 --subflag2 arg1 arg2
```

从上面可以看出，如果 `CommandLine` 包含了子命令，可以理解为本身也就没了参数，因为 `CommandLine` 的第一个参数即是子命令的名称，而之后的参数要解析为子命令的选项参数了。

现在，子命令的实现就变得非常简单了，创建一个新的 `FlagSet`，将 `CommandLine` 中的参数按前面介绍的流程重新处理一下。

第一步，获取 `CommandLine.Arg(0)`，检查是否存在相应的子命令。
```go
func main() {
	flag.Parse()
	if h {
		flag.Usage()
		return
	}

	cmdName := flag.Arg(0)
	switch cmdName {
	case "list":
		_ = list.Exec(cmdName, flag.Args()[1:])
	case "install":
		_ = install.Exec(cmdName, flag.Args()[1:])
	}
}
```

子命令的实现定义在另外一个包中，以 `list` 命令为例。 代码如下：

```go
var flagSet *flag.FlagSet

var origin string

func init() {
	flagSet = flag.NewFlagSet("list", flag.ExitOnError)
	val := newStringEnumValue("installed", &origin, []string{"installed", "local", "remote"})
	flagSet.Var(
		val, "origin",
		"the origin of version information, such as installed, local, remote",
	)
}
```

上面的代码中，定义了 `list` 子命令的 `FlagSet`，并在 `Init` 方法为其增加了一个选项 `Flag`，`origin`。

`Run` 函数是真正执行业务逻辑的代码。

```go
func Run(args []string) error {
	if err := flagSet.Parse(args); err != nil {
		return err
	}

	fmt.Println("list --oriign", origin)
	return nil
}
```

最后的 `Exec` 函数组合 `Init` 和 `Run` 函数，已提供给 `main` 调用。

```go
func Run(name string, args []string) error {
	Init(name)
	if err := Run(args); err != nil {
		return err
	}

	return nil
}
```

命令行的解析完成，如果子命令还有子命令，处理的逻辑依然相同。接下来的工作，就可以开始在 `Run` 函数中编写业务代码了。

# Go 命令

现在，阅读下 Go 命令的实现代码吧。

由于大佬们写的代码是基于 flag 包实现纯手工打造，没用任何的框架，在可读性上会有点差。

源码位于 [go/src/cmd/go/cmd/main.go](https://github.com/golang/go/blob/master/src/cmd/go/main.go) 下，通过 `base.Go` 变量初始化了 Go 支持的所有命令，如下：

```go
base.Go.Commands = []*base.Command{
	bug.CmdBug,
	work.CmdBuild,
	clean.CmdClean,
	doc.CmdDoc,
	envcmd.CmdEnv,
	fix.CmdFix,
	fmtcmd.CmdFmt,
	generate.CmdGenerate,
	modget.CmdGet,
	work.CmdInstall,
	list.CmdList,
	modcmd.CmdMod,
	run.CmdRun,
	test.CmdTest,
	tool.CmdTool,
	version.CmdVersion,
	vet.CmdVet,

	help.HelpBuildmode,
	help.HelpC,
	help.HelpCache,
	help.HelpEnvironment,
	help.HelpFileType,
	modload.HelpGoMod,
	help.HelpGopath,
	get.HelpGopathGet,
	modfetch.HelpGoproxy,
	help.HelpImportPath,
	modload.HelpModules,
	modget.HelpModuleGet,
	modfetch.HelpModuleAuth,
	modfetch.HelpModulePrivate,
	help.HelpPackages,
	test.HelpTestflag,
	test.HelpTestfunc,
}
```

无论是 `go` 命令，还是它的子命令，都是 `*base.Command` 类型。可以看一下 `*base.Command` 的定义。

```go
type Command struct {
	Run func(cmd *Command, args []string)

	UsageLine string
	Short string
	Long string

	Flag flag.FlagSet

	CustomFlags bool

	Commands []*Command
}
```

主要的字段有三个，分别是 `Run`，主要负责业务逻辑的处理，`FlagSet`，负责命令行的解析，以及 `[]*Command`， 所支持的子命令。

再来看看 `main` 函数中的核心逻辑。如下：

```go
BigCmdLoop:
for bigCmd := base.Go; ; {
	for _, cmd := range bigCmd.Commands {
		// ...
		// 主要逻辑代码
		// ...
	}

	// 打印帮助信息
	helpArg := ""
	if i := strings.LastIndex(cfg.CmdName, " "); i >= 0 {
		helpArg = " " + cfg.CmdName[:i]
	}
	fmt.Fprintf(os.Stderr, "go %s: unknown command\nRun 'go help%s' for usage.\n", cfg.CmdName, helpArg)
	base.SetExitStatus(2)
	base.Exit()
}
```

从最顶层的 `base.Go` 开始，遍历 Go 的所有子命令，如果没有相应的命令，则打印帮助信息。

省略的那段主要逻辑代码如下：

```go
for _, cmd := range bigCmd.Commands {
	// 如果找不到命令，继续下次循环
	if cmd.Name() != args[0] {
		continue
	}
	// 检查是否存在子命令
	if len(cmd.Commands) > 0 {
		// 将 bigCmd 设置为当前的命令
		// 比如 go tool compile，cmd 即为 compile
		bigCmd = cmd
		args = args[1:]
		// 如果没有命令参数，则说明不符合命令规则，打印帮助信息。
		if len(args) == 0 {
			help.PrintUsage(os.Stderr, bigCmd)
			base.SetExitStatus(2)
			base.Exit()
		}
		// 如果命令名称是 help，打印这个命令的帮助信息
		if args[0] == "help" {
			// Accept 'go mod help' and 'go mod help foo' for 'go help mod' and 'go help mod foo'.
			help.Help(os.Stdout, append(strings.Split(cfg.CmdName, " "), args[1:]...))
			return
		}
		// 继续处理子命令
		cfg.CmdName += " " + args[0]
		continue BigCmdLoop
	}
	if !cmd.Runnable() {
		continue
	}
	cmd.Flag.Usage = func() { cmd.Usage() }
	if cmd.CustomFlags {
		// 解析参数和选项 Flag
		// 自定义处理规则
		args = args[1:]
	} else {
		// 通过 FlagSet 提供的方法处理
		base.SetFromGOFLAGS(cmd.Flag)
		cmd.Flag.Parse(args[1:])
		args = cmd.Flag.Args()
	}

	// 执行业务逻辑
	cmd.Run(cmd, args)
	base.Exit()
	return
}
```

主要是几个部分，分别是查找命令，检查是否存在子命令，选项和参数的解析，以及最后是命令的执行。

通过 `cmd.Name() != args[0]` 判断是否查找到了命令，如果找到则继续向下执行。

通过 `len(cmd.Commands)` 检查是否存在子命令，存在将 `bigCmd` 覆盖，并检查是否符合命令行是否符合规范，比如检查 `len(args[1:])` 如果为 0，则说明传入的命令行没有提供子命令。如果一切就绪，通过 `continue` 进行下一次循环，执行子命令的处理。

接着是命令选项和参数的解析。可以自定义处理规则，也可以直接使用 `FlagSet.Parse` 处理。

最后，调用 `cmd.Run` 执行逻辑处理。

# 总结

本文介绍了 Go 中如何通过 flag 实现子命令，从 `FlagSet` 这个结构体讲起，通过 flag 包中默认提供的 `CommandLine` 梳理了 `FlagSet` 的处理逻辑。在基础上，实现了子命令的相关功能。

本文最后，分析了 Go 源码中 `go` 如何使用 flag 实现。因为是纯粹使用 flag 包裸写，读起来稍微有点难度。本文只算是一个引子，至少帮助大家在大的方向不至于迷路，里面更多的细节还需要自己挖掘。

博文地址：[Go 命令行解析 flag 包之通过子命令实现看 go 命令源码](https://www.poloxue.com/posts/2019-11-30-golang-flag-sub-commandline)
