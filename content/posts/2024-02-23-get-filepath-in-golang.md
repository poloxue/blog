---
title: "如何正确处理 Go 项目中关于文件路径的问题"
date: 2024-02-23T08:00:00+08:00
draft: false
comment: true
description: "在使用 Go 开发项目时，估计有不少人遇到过无法正确处理文件路径的问题，特别是刚从如 PHP、python 这类动态语言转向 Go 的朋友，已经习惯了通过相对源码文件找到其他文件。这个问题能否合理解决，不仅关系到程序的可移植性，还直接影响到程序的稳定性和安全性。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-23-get-filepath-in-golang-01.png)

> 嗨，大家好！我是波罗学。本文是系列文章 Go 技巧第十九篇，系列文章查看：[Go 语言技巧](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MzE2NTY2MA==&action=getalbum&album_id=3291066778475053060#wechat_redirect)。

在使用 Go 开发项目时，估计有不少人遇到过无法正确处理文件路径的问题，特别是刚从如 PHP、python 这类动态语言转向 Go 的朋友，已经习惯了通过相对源码文件找到其他文件。这个问题能否合理解决，不仅关系到程序的可移植性，还直接影响到程序的稳定性和安全性。

本文将尝试从简单到复杂，详细介绍 Go 中获取路径的不同方法及应用场景。

## 引言

首先，为什么要获取文件路径？

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-23-get-filepath-in-golang-02.png)

一般来说，程序在运行时必须准确地读取相关的配置和资源以顺利启动。确定这些信息的存储位置，即获取文件路径，成为了正确访问这些信息的首要步骤，对于构建稳定可靠的应用程序而言至关重要。

其次，为什么从动态语言转到 Go，容易被这个问题困扰？

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-23-get-filepath-in-golang-03.png)

与 Go（一种静态语言）相比，动态语言通过直接解释脚本文件而执行的。这一机制使得动态语言在路径获取方面更为直观和易懂。然而，Go语言将源代码编译成独立的二进制可执行文件，这导致可执行文件与源代码间缺乏直接的联系。

为了简化调试过程，Go 通过 `go run` 命令提供了一种类似动态语言直接执行源代码的便捷方式，实质上是将构建和运行步骤合二为一。这个过程中，会生成一个临时可执行文件，但这个文件不是存在当前工作目录中，这又为理解上带来额外的挑战。

如果想找到这个文件，可通过 `go run -work` 保留文件，通过 `os.Args[0]` 确认文件路径。

```go
func main() {
	fmt.Println(os.Args[0])
}
```

输出：
```bash
$ go run -work main.go
WORK=/var/folders/0b/v4r1lzyj0n566qgd8dt_km4c0000gn/T/go-build1458488796
/var/folders/0b/v4r1lzyj0n566qgd8dt_km4c0000gn/T/go-build1458488796/b001/exe/main
```

可执行文件就是位于 `$WORK/b001/exe/` 的 `main` 文件。

若你习惯于动态语言中获取路径的做法，在 Go 中通过相对于可执行文件的路径来定位其他文件，使用 `go run` 调试的时候，就可能会引起一定的困惑。

下面开始进入正题，详细 Go 中的文件路径的不同获取方式吧。

## 相对于执行文件获取路径

之前提到了那么多在 Go 中获取可执行文件路径时可能导致的问题，我们就先从如何获取当前执行文件的路径开始吧。

我将介绍实现这个目标的两种方式。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-23-get-filepath-in-golang-04.png)

### 命令行参数 `os.Args[0]`

第一种方式是通过命令行参数 `os.Args[0]`。`os.Args` 是一个字符串切片，包含启动程序时传递给它的命令行参数。`os.Args[0]` 是这个切片的第一个元素，通常表示程序的执行文件路径。引言部分的演示示例，我就是通过这种方式获取执行文件的路径的。

这个方式缺点是，依赖于可执行文件是被调用的方式，它可能是一个相对路径、一个绝对路径，或者仅仅是程序名。

于是，为了保险起见，我们可通过 `exec.LookPath` 对 `os.Args[0]` 做一个处理。

```go
fmt.Println(exec.LookPath(os.Args[0]))
```

这个函数的作用是，输入参数 `filename` 中如果包含如 `/` 字符，直接返回 `filename`，否则会从 `PATH` 环境变量中寻找名为 `filename` 的可执行文件。这就解决了仅仅通过程序名调用无法获取文件路径的问题。

> 我是在 MacOS 上测试的，这段逻辑是在 lp_unix.go 文件中，window 应该是不同的逻辑，windows 的文件路径分隔符和类 unix 不同，或者也有其他复杂逻辑。

另外，它获取到的可能是相对路径也可能是绝对路径。如果希望得到绝对路径，要通过 `filepath.Abs` 处理下。

```go
exePath, _ := exec.LookPath(os.Args[0])
fmt.Println(filepath.Abs(exePath))
```

但这种不是最优的方式，明显是绕的远了。 我提这个方法是为了顺便介绍下 `exec.LookPath` 和 `filepath.Abs` 这两个函数。

### 使用 `os.Executable`

获取当前 Go 程序的执行文件路径最优的解法是，使用 `os.Executable` 函数。这个方法会返回可执行文件的绝对路径。

```go
fmt.Println(os.Executable()) // 
```

输出:

```bash
$ go run main.go
/var/folders/0b/v4r1lzyj0n566qgd8dt_km4c0000gn/T/go-build305466852/b001/exe/main <nil>
```

这个值在 go 启动时，运行时自动解析到内存的值，而调用 `os.Executable` 实际就是直接从这个变量中获取，没有额外的处理。

它的性能相对于前面的通过几个函数组合实现的方式，肯定是吊打前者。

但，这两种方式都没有解决一个问题：如果执行文件是符号链接，不会返回真正的可执行文件。

### 符号链接

我们可通过使用 `filepath.EvalSymlinks` 来获取符号链接实际指向的路径。

```go
realPath,  _:= filepath.EvalSymlinks(exePath)
fmt.Println("Real path of executable:", realPath)
```

## 兼容 `go run` 与 `go build`

讲了那么多关于获取当前执行文件路径的方案，但如何解决由 `go run` 临时文件产生的问题呢？

我的建议是，换个思路，不要把拘泥在相对于可执行文件定位其他文件路径这一个方向上。我在网上看到过通过判断是否是 `go run` 运行实现的适配方案。

大概意思是，通过判断执行文件的运行目录或手动添加环境变量标识当前位于 `go run` 运行模式。如果处理 `go run` 模式下，我们再通过相对于源码文件位置定位其他文件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-23-get-filepath-in-golang-05.png)

尝试实现下吧。

```go
// isGoRun 检查当前是否处于 go run 模式
func isGoRun() bool {
	// 检查环境变量（如果你选择设置一个特定的环境变量来标识）
	if _, ok := os.LookupEnv("GO_RUN_MODE"); ok {
		return true
	}
}
```

或者是

```go
func isGoRun() bool {
	// 或者通过分析 executable 路径的特征来判断
	exePath, err := os.Executable()
	if err != nil {
		fmt.Println("Error getting executable path:", err)
		return false
	}

	// 示例中仅仅检查路径是否包含临时目录特征，实际情况可能需要更复杂的逻辑
	return exePath[:5] == "/var/" {
}
```

而在入口函数 `main` 中，通过 `runtime.Caller(0)` 获取源码文件路径。

```go
func EntryPath() string {
	if IsGoRun() {
		_, file, _, ok := runtime.Caller(0)
		if ok {
			return filepath.Dir(file)
		}
	} else {
		path, _ := os.Executable()
		return filepath.Dir(path)
	}
	return "./"
}

func main() {
	configPath := filepath.Join(EntryPath(), "config.json")
	fmt.Println("ConfigPath:", configPath)
}
```

除了那个获取源码文件位置的函数 `runtime.Caller`，这个代码并不复杂。`runtime.Caller` 函数用于获取当前函数的调用栈信息。

它的函数签名，如下所示：

```go
func Caller(skip int) (pc uintptr, file string, line int, ok bool)
```

返回信息有调用者（main 函数）的程序计数器（PC）、文件名、代码行号、一个布尔值，布尔值表示获取信息是否成功。我们关心的是源码文件路径，`runtime.Caller` 返回的文件名可以用来确定当前执行代码的位置。

看到这里，不知道是不是有人发出疑问，竟然通过能定位源码文件位置，为什么还要另外一种方式。这是源码文件的位置不会因执行文件的移动而变动。举例来说，如果 `main.go` 文件在 `~/Users/poloxue/` 下构建出 `main` 执行文件。我将其移动到其他目录，甚至是服务器上，它的路径依然是 `/Users/poloxue/main.go`。

现在，即使在 `go run` 模式下，依然能正确定位其他文件的路径了。

这种方式看起来挺不错的，但我不推荐。我的建议是，为项目定义清晰明确的规则来管理配置和资源文件的路径。

## 定义明确的路径规则

常见的是用绝对路径规则指定配置和资源文件路径，如 Linux 或其他类 Unix 系统有一套 XDG 基准规则（XDG Base Directory Specification），有兴趣可了解下。

或者是另一套更常见日常项目中的方案，通过环境变量或其他方式设置固定的项目根目录或工作目录，而其他文件路径皆相对于这个固定不变目录的位置。

```
$RootDir/config.yaml
$RootDir/logs/
$RootDir/resources/
$RootDir/static
```

实际上，这种方式更常见于平时的项目中。无论可执行文件被放在什么路径下，都不会对其他文件的路径位置产生影响。

如果希望文件路径支持自定义，可在配置中提供路径配置项，或通过命令行选项的方式传递。

```toml
log_path = "/var/log/"
```

或

```bash
$ go run main.go --config-path "./config.toml"
```

如果觉得每次 `go run` 都要带上环境变量麻烦，可提前设置环境变量

```bash
export ROOTDIR=`pwd`
```

我们也可以在 IDE 中设置项目级别的环境变量。

亦或是提供默认值，如果 ROOTDIR 为空，默认项目根目录为 `./`，即当前路径，

```bash
# ROOTDIR=./ go run main.go
$ go run main.go
```

如果是运行在 Docker 中，可通过 `WORKDIR` 指定工作目录，问题也变得简单很多。

## 总结

在 Go 项目中正确处理文件路径是确保程序可移植性、稳定性和安全性的关键。与动态语言不同，Go编译成二进制可执行文件，使得直接关联源码和运行时文件变得复杂。

本文介绍了多种获取文件路径的方法，包括 `os.Args[0]`、`exec.LookPath`、`filepath.Abs`和 `os.Executable`，并讨论了如何通过判断是否是 `go run` 运行来兼容 `go run` 和`go build` 的路径问题。

最后，建议定义清晰的规则管理配置和资源文件路径，使用环境变量或配置项指定路径，避免依赖于可执行文件位置，以求提高 Go 项目的健壮性。

感谢阅读，欢迎关注我的更多文章。

