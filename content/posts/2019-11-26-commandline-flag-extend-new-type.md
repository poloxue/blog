---
title: "Go 命令行解析 flag 包之扩展新类型"
date: 2019-11-26T16:08:21+08:00
draft: false
comment: true
tags: ["Golang"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-26-commandline-flag-extend-new-type-01.png)

[上篇文章](https://www.poloxue.com/posts/2019-11-23-commandline-tool-flag-in-golang/) 说到，flag 支持的类型有布尔类型、整型（int、int64、uint、uint64）、浮点型（float64）、字符串（string）和时长（duration）。

一般情况下，flag 内置类型能满足绝大部分的需求，但某些场景，我们要自定义解析规则。一个优秀的库肯定要支持扩展的。

本文将介绍如何为 flag 扩展一个新的类型支持？

# 扩展目标

上文中，假设我们在开发一个 `gvg` 命令行工具。它其中的 `list` 子命令支持获取 Go 的版本列表。但 Go 版本来源信息有多处，比如 `installed`（已安装）、`local`（本地仓库）和 `remote`（远程仓库）。

查看下 `list` 的帮助信息，如下：

```bash
NAME:
   gvg list - list go versions

USAGE:
   gvg list [command options] [arguments...]

OPTIONS:
   --origin value  the origin of version information , such as installed, local, remote (default: "installed")
```

从帮助信息中可知，`list` 支持一个 `Flag` 选项，`--origin`。它用于指定版本信息的来源，允许值的范围是 `installed`、`local` 和 `remote`。

如果要求不严格，用 `StringVar` 也可以实现。但问题是，使用 `String`，即使输入不在指定范围也能成功解析，不够严谨。虽说在获取后也可以检查，但还是不够灵活、可配置型也差。

接下来，我们要实现一个新的类型的 `Flag`，使选项的值必需在指定范围，否则要给出一定的错误提示信息。

# 实现思路

如何展一个新类型呢？

参考 flag 包内置类型的实现思路，如 `flag.DurationVar`。`Duration` 不是基础类型，解析结果是存放到了 `time.Duration` 类型中，可能有参考价值。

我们进入 `flag.DurationVar` 查看源码，如下：

```go
func DurationVar(p *time.Duration, name string, value time.Duration, usage string) {
	CommandLine.Var(newDurationValue(value, p), name, usage)
}
```

通过 `newDurationValue` 创建了一个类型为 `durationValue` 的变量，并传入到了 `CommandLine.Var` 方法中。

如果继续往下追，会根据 Value 创建一个 `Flag` 变量。 如下：

```go
func (f *FlagSet) Var(value Value, name string, usage string) {
	flag := &Flag{name, usage, value, value.String()}
	...
}
```

从 `Var` 的定义可以看出，它的第一个参数类型是 `Value` 接口类型，也就说，durationValue 是实现了 `Value` 接口的类型。

注意，源码中出现的 `FlagSet` 可以先忽略，它是下篇介绍子命令时重点关注的对象。

看下 `Value` 的定义，如下：

```go
type Value interface {
	String() string
	Set(string) error
}
```

那么，`durationValue` 的实现代码如何？

```go
// 传入参数分别是默认值和获取 Flag 值的变量地址
func newDurationValue(val time.Duration, p *time.Duration) *durationValue {
	// 将默认值设置到 p 上
	*p = val
	// 使用 p 创建新的类型，保证可以获取到解析的结果
	return (*durationValue)(p)
}

// Set 方法负责解析传入的值
func (d *durationValue) Set(s string) error {
	v, err := time.ParseDuration(s)
	if err != nil {
		err = errParse
	}
	*d = durationValue(v)
	return err
}

// 获取真正的值
func (d *durationValue) String() string { return (*time.Duration)(d).String() }
```

核心在两个地方。

一个是创建新类型变量时，要使用传入的变量地址创建新类型变量，以实现将解析结果放到其中，让前端能获取到，二是 `Set` 方法中实现命令行传入字符串的解析。


# 逻辑梳理

看完上个小节，基本已经了解如何扩展一个新类型了。本质是是实现 `Value` 接口。

再看下之前提到的几个变量，分别是存放解析结果的指针、解析命令行输入的 `Value` 和表示一个选项的 `Flag`。对应于 `flag.DurationVar`，这个变量的类型分别是 `*time.Duration`、`durationValue` 和 `Flag`。

比如有 `duration=1h`，大致流程是首先从 `os.Args` 获取参数，按规则解析出选项名称 `duration`，查找是否存在名称为 `duration` 的 `Flag`，如果存在，使用 `Flag.Value.Set` 解析 `1h`，如果不满足 `duration` 的要求，将给出错误提示。

# 实现新类型

现在实现文章开头要求的目标。

新类型定义如下：

```go
type stringEnumValue struct {
	options []string
	p   *string
}
```

我们创建了新类型 - `stringEnumValue`，即字符串枚举。它有 `options` 和 `p` 两个成员，`options` 指定一定范围的值，`p` 是 `string` 指针，保存解析结果的变量的地址。

下面定义创建 `StringEnumValue` 变量的函数 `newStringEnumValue`，代码如下：

```go
func newStringEnumValue(val string, p *string, options []string) *StringEnumValue {
	*option = val
	return &stringEnumValue{options: options, p: p}
}
```

除了 `val` 和 `p` 两个必要的输入外，还有一个 `string` 切片类型的数，名为 `options`，它用于范围的限定。而函数主体，首先设置默认值，然后使用 `options` 和 `p` 创建变量返回。

`Set` 是核心方法，解析命令行传入字符串。代码如下：

```go
func (s *StringEnumValue) Set(v string) error {
	for _, option := range s.options {
		if v == option {
			*(s.p) = v
			return nil
		}
	}

	return fmt.Errorf("must be one of %v", s.options)
}
```

循环检查输入参数 `v` 是否满足要求。定义如下：

最后是 `String()` 方法，
```go
func (s *StringEnumValue) String() string {
	return *(s.p)
}
```

返回 `p` 指针中的值。前面分析实现思路时，`Flag` 在设置默认值时就调用了它。

# 使用 StringEnumValue

直接看代码吧。如下：

```go
var origin string
func init() {
	flag.Var(
		newStringEnumValue(
			"installed", 	// 默认值
			&origin,
			[]string{"installed", "local", "remote"},
		),
		"origin",
		`the origin of version information, such as installed, local, remote (default: "installed")`,
	)
}

func main() {
	flag.Parse()
	fmt.Println(option)
}
```

重点就是 `flag.Var(newStringEnumValue(...)，...)`。如果觉得有点啰嗦，希望和其他类型新建过程相同，在这个基础上可以再包装。代码如下：

```go
func StringEnumVar(p *string, name string, options []string, defVal string, usage string) {
	flag.Var(newStringEnumValue(defVal, p, options), name, usage)
}
```

编译测试下，结果如下：

```bash
$ gvg --origin=any
invalid value "any" for flag -origin: must be one of [installed local remote]
Usage of gvg:
  -origin value
  	the origin of version information, such as installed, local, remote (default installed)
$ gvg --origin=remote
origin remote
```

# 总结

本文介绍了如何为 flag 扩展一个类型支持，通过分析源码理清实现思路。最后创建了一个只接收指定范围值的 Value。

博文地址：[Go 命令行解析 flag 包之扩展新类型](https://www.poloxue.com/2019-11-26-commandline-flag-extend-new-type)
