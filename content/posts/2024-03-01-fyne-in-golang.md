---
title: "2024 03 01 Fyne in Golang"
date: 2024-02-29T18:42:00+08:00
draft: true
comment: true
description: ""
---

今天，推荐一个 Go 实现的 GUI 库 - fyne。

Go 一直以来都没有一个标准 GUI 库，Go 官方也没有提供。在 Go 语言的几个 GUI 库中，Fyne 算是最出色的，它有着简洁的API、支持跨平台能力，且高度可扩展。

本文将从浅到深逐步对 Fyne 展开介绍，希望对大家快速上手这个 GUI 框架有所帮助。

## 安装 fyne

开始前，确保你已安装Go。如果是 MacOS X 系统，请确认安装了 xcode。使用 `go get` 命令安装 Fyne。

```bash
$ mkdir hellofyne
$ cd helloyfyne
$ go mod init hellofyne
$ go get fyne.io/fyne/v2@latest
$ go install fyne.io/fyne/v2/cmd/fyne@latest
```

安装完成后，可从 [setup](https://geoffrey-artefacts.fynelabs.com/github/andydotxyz/fyne-io/setup/latest/) 下载 fyne 的安装检测程序检查下。

检测结果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-02-v1.png)

如果想立刻查看 fyne 提供的演示案例，通过命令检查：

```bash
$ go run fyne.io/fyne/v2/cmd/fyne_demo@latest
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-04-v1.png)

看起来，这里面的案例还是不够丰富的。

到此，准备工作结束！

## 创建第一个应用

由于 Go 简洁的语法和 fyne 的设计，使用 fyne 创建一个 GUI 应用异常简单。

以下是一个创建基础窗口的例子：

```go
package main

import (
    "fyne.io/fyne/v2/app"
    "fyne.io/fyne/v2/container"
    "fyne.io/fyne/v2/widget"
)

func main() {
    a := app.New()
    w := a.NewWindow("Hello Fyne")
    w.SetContent(widget.NewLabel("Welcome to Fyne!"))
    w.ShowAndRun()
}
```

这段代码创建了一个包含标签的窗口。

通过 `app.New()` 初始化一个 Fyne 应用实例。然后，创建一个标题为 "Hello Fyne" 的窗口，并设置内容为包含 "Welcome to Fyne!" 文本标签。最后，通过`w.ShowAndRun()`显示窗口并启动应用的事件循环。

fyne 中窗口的默认大小是其包含的内容决定，也可预置大小。

如在创建窗口后，提供 `Resize` 重新调整大小。

```go
w.Resize(fyne.NewSize(100, 100))
```

还有，一个 app 下可以有多个窗口的，示例代码：

```go
btn := widget.NewButton("Create a widnow", func() {
	w2 := a.NewWindow("Window 02")
	w2.Resize(fyne.NewSize(200, 200))
	w2.Show()
})

w.SetContent(btn)
```

我们创建一个按钮，它的点击事件是创建一个新的窗口并显示。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-08.gif)

## 布局和控件

布局和控件是 GUI 框架中必不可少的组件。Fyne 提供了多种布局管理器和标准的 UI 控件，支持创建更复杂的界面。

### 布局管理

Fyne 中的布局的实现位于 `container` 包中。

它提供了多种不同布局方式安排窗口中的元素。例如，例如最基本的 `HBox` 水平布局和 `VBox` 垂直布局。

通过 `HBox` 创建水平布局：

```go
w.SetContent(container.NewHBox(widget.NewButton("Left", func() {
	fmt.Println("Left button clicked")
}), widget.NewButton("Right", func() {
	fmt.Println("Right button clicked")
})))
```

显示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-09.png)

同理，`VBox` 创建垂直布局：

```go
w.SetContent(container.NewVBox(widget.NewButton("Top", func() {
	fmt.Println("Top button clicked")
}), widget.NewButton("Bottom", func() {
	fmt.Println("Bottom button clicked")
})))
```

显示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-10.png)


Fyne 除了基础的水平（`HBoxLayout`）和垂直（`VBoxLayout`）布局，还提供了`GridLayout`、`FormLayout`等高级布局方式。`GridLayout`可以将元素均匀地分布在网格中，而`FormLayout`适用于创建表单，自动排列标签和字段。这些灵活的布局选项支持创建更复杂和功能丰富的GUI界面。

在 Fyne 示例中可以找到更多布局的内容：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-11.png)

或者查看官方文档：[Layout List](https://docs.fyne.io/explore/layouts)。

### 更多控件

前面的演示案例中，用到了两个控件：Label 和 Button，Fyne 还支持其他多种控件，它们都为于 `widget` 包中。

常见控件一览：

```go
label := widget.NewLabel("Label")
button := widget.NewButton("Button", func() {})
entry := widget.NewEntry()
entry.SetPlaceHolder("Entry")
check := widget.NewCheck("Check", func(bool) {})
radio := widget.NewRadioGroup([]string{"Option 1", "Option 2"}, func(string) {})
selectEntry := widget.NewSelectEntry([]string{"Option A", "Option B"})
progressBar := widget.NewProgressBar()
slider := widget.NewSlider(0, 100)
combo := widget.NewSelect([]string{"Option A", "Option B", "Option C"}, func(string) {})
formItem := widget.NewFormItem("FormItem", widget.NewEntry())
form := widget.NewForm(formItem)
accordion := widget.NewAccordion(widget.NewAccordionItem("Accordion", widget.NewLabel("Content")))
tabs := container.NewAppTabs(
	container.NewTabItem("Tab 1", widget.NewLabel("Content 1")),
	container.NewTabItem("Tab 2", widget.NewLabel("Content 2")),
)

// 弹出对话框示例按钮
dialogButton := widget.NewButton("Show Dialog", func() {
	dialog.ShowInformation("Dialog", "Dialog Content", w)
})

content := container.NewVScroll(container.NewVBox(
	label, button, entry, check, radio, selectEntry, progressBar, slider,
	combo, form, accordion, tabs, dialogButton,
))
w.SetContent(content)
```

演示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-12.gif)

## 自定义控件和数据绑定

要在实际项目中使用 Fyne，基本上是要自定义控件或者使用数据绑定功能的。自定义控件用于创建符合需求的控件，而数据绑定用于帮助我们轻松地创建动态 UI。

### 自定义控件

如果要在 fyne 上实现自定义控件，涉及定义控件的绘制方法和布局逻辑。

首先，自定义控件要实现两个接口：`fyne.Widget` 和 `fyne.WidgetRenderer`。

首先，`fyne.Widget` 的定义如下所示：

```go
type Widget interface {
	CanvasObject
	CreateRenderer() WidgetRenderer
}
```

而其中返回的就是 `WiddgetRenderer`，用于定义控件布局渲染的逻辑。

```go
type WidgetRenderer interface {
	Destroy()
	Layout(Size)
	MinSize() Size
	Objects() []CanvasObject
	Refresh()
}
```

假设，现在我要实现一个简单的控件，一个类似 Label 的控件。

如下是这个控件类型的定义：

```go
type CustomLabel struct {
    widget.BaseWidget
    Text              string
}
```

它继承了 `wiget.BaseWidget` 的基本控件实现，我们只要实现 `CreateRenderer` 方法即可。

定义 `customWidgetRenderer` 类型：

```go
type customWidgetRenderer struct {
    text  *canvas.Text // 使用canvas.Text来绘制文本
    owner *CustomWidget
}
```

### 数据绑定

Fyne 的数据绑定功能允许控件直接与数据源连接，数据的任何更改都会自动反映在UI上，反之亦然。

## 结语

Fyne 以其简单、强大和跨平台的特性，为Go语言开发现代GUI应用提供了一个优秀的选择。无论你是初学者还是有经验的开发者，Fyne都值得一试。随着对 Fyne 的深入，你将能够更加灵活地构建出符合需求的应用，享受Go语言开发GUI带来的乐趣。

