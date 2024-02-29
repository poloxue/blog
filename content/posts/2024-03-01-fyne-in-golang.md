---
title: "2024 03 01 Fyne in Golang"
date: 2024-02-29T18:42:00+08:00
draft: true
comment: true
description: ""
---

一直好奇 Go 有没有什么好用的 GUI 框架？经过一番搜索，发现了 fyne 这个框架，也不知道好不用。为快速了解和使用它，我基于它的文档和 example 以尽量简化的方式写成了这篇文章。

## 快速开始

fyne 是一个跨平台的 GUI 框架，它可以运行在 Windows、macOS X、 Linux、BSD、Android 和 IOS 平台上，即用这个框架，是可以开发 APP 的。

### 安装

在 MacOS X 上，它的安装依赖 XCode 和 Go 语言环境。在确认基本依赖没有问题后，通过如下命令安装和初始化一个项目：

```go
$ mkdir hellofyne
$ cd helloyfyne
$ go mod init hellofyne
$ go get fyne.io/fyne/v2@latest
$ go install fyne.io/fyne/v2/cmd/fyne@latest
```

安装完成后，从地址 [setup](https://geoffrey-artefacts.fynelabs.com/github/andydotxyz/fyne-io/setup/latest/) 下载 fyne 的安装检测程序检查安装结果。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-03.png)

检测结果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-02-v1.png)

如果想立刻查看 fyne 提供的演示案例，通过命令检查：

```bash
$ go run fyne.io/fyne/v2/cmd/fyne_demo@latest
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-04-v1.png)

看起来，这其中的案例还是挺简陋的。

准备工作结束！

### Hello World

接下来就是开发第一个 hellofyne 程序，从 `app.New` 开始。

示例代码：

````go
package main

import (
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/widget"
)

func main() {
	a := app.New()
	w := a.NewWindow("Hello World")

	w.SetContent(widget.NewLabel("Hello Fyne!"))
	w.ShowAndRun()
}
``````

通过 `app` 创建应用，它是核心，基于 `app` 创建窗口 - `app.NewWindow`，而窗口上布局的内容 widget，由 `widget` 包提供，如 `widget.NewLabel` 创建一个标签 Label。 

运行程序：

```bash
$ go mod tidy
$ go run .
```

效果图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-06.png)

其它语言的支持，如中文，要单独设置字体，如将 "Hello Fyne" 修改为 "你好，Fyne"。通过环境变量 `FYNE_FONT` 指定字体文件，如下命令运行：

```
FYNE_FONT=~/SourceHanSerifSC-VF.ttf go run .
```

效果图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-05-v1.png)

如果找不到测试字体，可从 [SourceHanSerifSC](https://github.com/adobe-fonts/source-han-serif/tree/release/?tab=readme-ov-file) 下载一个 ttf 文件。

最后的 `w.ShowAndRun` 显示和开启事件循环，一个 app 只能调用一次 `Run`，`w.ShowAndRun` 是由两部分组成的，分别是 `w.Show` 和 `app.Run`。

### 更新 widget 内容

前面演示了一个 fyne 的 "Hello World" 程序。现在，参考官网案例，让这个 `Hello World` 文字变动起来，显示当前时间。

首先，定义一个类型为 Label 的变量 `clock`，用于显示时间。

```go
clock := wiget.NewLabel("")
w.SetContent(clock)
```

为了不能干扰程序的渲染内容，我们可直接开启一个 goroutine 定时更新 Label 中的文字。

```go
go func() {
  for range time.Tick(time.Second) {
    formatted := time.Now().Format("Time: 03:04:05")
    clock.SetText(formatted)
	}
}
```

效果图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-07.gif)

完整代码查看官方文档：[Updating Content in your GUI](https://docs.fyne.io/started/updating)

### Window

fyne 的 Window 默认大小是由显示的内容决定的，如前面 Window 中只有一个 Label，显示内容是 "Hello Fyne" 或时间，它的框架只有内容那么大。

如果希望预置 Window 大小，如将 Window 的宽高皆设为 100，实现代码如下：

```go
w.Resize(fyne.NewSize(100, 100))
```

还有，虽说一个程序中只有一个 `app`，但可以有多个 `window`。

```go
a := app.New()
w := a.NewWindow("Hello World")

w.SetContent(widget.NewLabel("Hello World!"))
w.Show()

w2 := a.NewWindow("Larger")
w2.SetContent(widget.NewLabel("More content"))
w2.Resize(fyne.NewSize(100, 100))
w2.Show()

a.Run()
```

这种情况下，就不建议使用 `Window` 调用 `ShowAndRun`了。

`Window` 的创建可以发生在任何时间，如某个应用要在某个按键点击事件时单独开启一个 `window` 显示数据，可通过类似如下代码直接新建一个 `Window`：

```go
w2.SetContent(widget.NewButton("Open new", func() {
	w3 := a.NewWindow("Third")
	w3.SetContent(widget.NewLabel("Third"))
	w3.Show()
}))
```

到这里，fyne 快速开始部分介绍结束。


