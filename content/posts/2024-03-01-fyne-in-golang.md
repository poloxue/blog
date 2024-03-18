---
title: "一个 Go 实现的跨平台 GUI 框架 Fyne"
date: 2024-03-07T08:00:00+08:00
draft: false
comment: true
description: "Go 一直以来都没有一个标准 GUI 库，Go 官方也没有提供。在 Go 实现的几个 GUI 库中，Fyne 算是最出色的，它有着简洁的API、支持跨平台能力，且高度可扩展。这也就是说，Fyne 是用来开发 App。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-01.png)

今天，推荐一个 Go 实现的 GUI 库 - fyne。

Go 官方也没有提供标准的 GUI 框架，在 Go 实现的几个 GUI 库中，Fyne 算是最出色的，它有着简洁的API、支持跨平台能力，且高度可扩展。这也就是说，Fyne 是可以开发 App。

本文将尝试介绍下 Fyne，希望对大家快速上手这个 GUI 框架有所帮助。我最近产生了不少想法，其中有些是对 GUI 有要求的，就想着折腾用 Go 实现，而不是用那些已经很流行和成熟的 GUI 框架。

在写这篇文章时，顺手搞了下它的中文版文档，文档查看 [www.poloxue.com/gofyne](https://www.poloxue.com/gofyne)，希望能帮助那些想继续深入这个框架的朋友。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-15.png)

## 安装 fyne

开始前，确保已成功安装 Go，如果是 MacOS X 系统，要确认安装了 xcode。

如下使用 `go get` 命令安装 Fyne。

```bash
$ mkdir hellofyne
$ cd helloyfyne
$ go mod init hellofyne
$ go get fyne.io/fyne/v2@latest
$ go install fyne.io/fyne/v2/cmd/fyne@latest
```

如果想立刻查看 fyne 提供的演示案例，通过命令检查：

```bash
$ go run fyne.io/fyne/v2/cmd/fyne_demo@latest
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-04-v1.png)

看起来，这里面的案例还是不够丰富的。

安装工作到此就完成了。Fyne 对不同系统有不同依赖，如果安装过程中遇到问题，细节可查看官方提供的[安装文档](https://www.poloxue.com/gofyne/docs/01-started/01-index/)。

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

布局和控件是 GUI 应用程序设计中必不可少的两类组件。Fyne 提供了多种布局管理器和标准的 UI 控件，支持创建更复杂的界面。

### 布局管理

Fyne 中的布局的实现位于 `container` 包中。它提供了多种不同布局方式安排窗口中的元素。最基本的布局方式有 `HBox` 水平布局和 `VBox` 垂直布局。

通过 `HBox` 创建水平布局的代码如下：

```go
w.SetContent(container.NewHBox(widget.NewButton("Left", func() {
	fmt.Println("Left button clicked")
}), widget.NewButton("Right", func() {
	fmt.Println("Right button clicked")
})))
```

显示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-09.png)

通过 `VBox` 创建垂直布局的例子。

```go
w.SetContent(container.NewVBox(widget.NewButton("Top", func() {
	fmt.Println("Top button clicked")
}), widget.NewButton("Bottom", func() {
	fmt.Println("Bottom button clicked")
})))
```

显示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-10.png)

Fyne 除了基础的水平（`HBoxLayout`）和垂直（`VBoxLayout`）布局，还提供了`GridLayout`、`FormLayout` 甚至是混合布局 C`ombinedLayout` 等高级布局方式。

如 `GridLayout`可以将元素均匀地分布在网格中，而`FormLayout`适用于创建表单，自动排列标签和字段。这些灵活的布局选项支持创建更复杂和功能丰富的GUI界面。

官方文档中的布局列表查看：[Layout List](https://www.poloxue.com/gofyne)。

### 更多控件

前面的演示案例中，用到了两个控件：Label 和 Button，Fyne 还支持其他多种控件，它们都为于 `widget` 包中。

我尝试在一份代码中展示出来，如下是常见控件一览：

```go
// 标签 Label
label := widget.NewLabel("Label")
// 按钮 Button
button := widget.NewButton("Button", func() {})
// 输入框 Entry
entry := widget.NewEntry()
entry.SetPlaceHolder("Entry")
// 复选框 Check
check := widget.NewCheck("Check", func(bool) {})
// 单选框 Check
radio := widget.NewRadioGroup([]string{"Option 1", "Option 2"}, func(string) {})
// 选择框
selectEntry := widget.NewSelectEntry([]string{"Option A", "Option B"}
// 进度条
progressBar := widget.NewProgressBar()
// 滑块
slider := widget.NewSlider(0, 100)
// 组合框
combo := widget.NewSelect([]string{"Option A", "Option B", "Option C"}, func(string) {})
// 表单项
formItem := widget.NewFormItem("FormItem", widget.NewEntry())
form := widget.NewForm(formItem)
// 手风琴
accordion := widget.NewAccordion(widget.NewAccordionItem("Accordion", widget.NewLabel("Content")))
// Tab 选择
tabs := container.NewAppTabs(
	container.NewTabItem("Tab 1", widget.NewLabel("Content 1")),
	container.NewTabItem("Tab 2", widget.NewLabel("Content 2")),
)

// 弹出对话框示例按钮
dialogButton := widget.NewButton("Show Dialog", func() {
	dialog.ShowInformation("Dialog", "Dialog Content", w)
})

// 滚动布局
content := container.NewVScroll(container.NewVBox(
	label, button, entry, check, radio, selectEntry, progressBar, slider,
	combo, form, accordion, tabs, dialogButton,
))
w.SetContent(content)
```

演示效果：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-12.gif)

## Fyne 中的自定义

如果在实际项目中使用 Fyne，基本上是要使用 Fyne 的自定义能力。Fyne 提供了自定义控件、布局和主题等。

### 自定义控件

fyne 是支持实现自定义控件的，这涉及定义控件的绘制方法和布局逻辑。我们主要是实现两个接口：`fyne.Widget` 和 `fyne.WidgetRenderer`。

`fyne.Widget` 的定义如下所示：

```go
type Widget interface {
	CanvasObject
	CreateRenderer() WidgetRenderer
}
```

`CreateRenderer` 方法返回的就是 `WiddgetRenderer`，用于定义控件渲染和布局的逻辑。

```go
type WidgetRenderer interface {
	Destroy()
	Layout(Size)
	MinSize() Size
	Objects() []CanvasObject
	Refresh()
}
```

这样拆分的目标是为将了控件的逻辑和 UI 绘制分离开来，在 `Widget` 中专注于逻辑，而 `WidgetRenderer` 中专注于渲染布局。

假设实现一个类似 Label 的控件，类型定义：

```go
type CustomLabel struct {
    widget.BaseWidget
    Text string
}
```

它继承了 `wiget.BaseWidget` 基本控件实现，`Text` 就是要 `Label` 显示的文本。还要给给 `CustomLabel` 实现 `CreateRenderer` 方法。

定义 `CustomLabel` 创建函数：

```go
func NewCustomLabel(text string) *CustomLabel {
	label := &CustomLabel{Text: text}
	label.ExtendBaseWidget(label)
	return label
}
```

`customWidgetRenderer` 类型定义如下：

```go
type customWidgetRenderer struct {
    text  *canvas.Text // 使用canvas.Text来绘制文本
    label *CustomLabel
}
```

实现 `CustomLabel` 的 `CreateRenderer` 方法。

```go
func (label *CustomLabel) CreateRenderer() fyne.WidgetRenderer {
	text := canvas.NewText(label.Text, theme.ForegroundColor())
	text.Alignment = fyne.TextAlignCenter

	return &customLabelRenderer{
		text:  text,
		label: label,
	}
}
```

构建 `Renderer` 变量，使用 `canvas` 创建 `Text` 文本框，为适配主题使用主题配置前景色。还有，设置文本居中显示。

而 `customLabelRenderer` 要实现 `WidgetRender` 接口定义的所有方法。

```go
func (r *customLabelRenderer) MinSize() fyne.Size {
	return r.text.MinSize()
}

func (r *customLabelRenderer) Layout(size fyne.Size) {
	r.text.Resize(size)
}

func (r *customLabelRenderer) Refresh() {
	r.text.Text = r.label.Text
	r.text.Color = theme.ForegroundColor() // 确保文本颜色更新
	r.text.Refresh()
}

func (r *customLabelRenderer) BackgroundColor() color.Color {
	return theme.BackgroundColor()
}

func (r *customLabelRenderer) Objects() []fyne.CanvasObject {
	return []fyne.CanvasObject{r.text}
}

func (r *customLabelRenderer) Destroy() {}
```

在 main 函数中，尝试使用这个控件。

```go
a := app.New()
w := a.NewWindow("Custom Label")

w.SetContent(NewCustomLabel("Hello"))
w.ShowAndRun()
```

显示的效果和 Label 控件是类似的。

### 其他自定义

其他自定义能力，如 Layout、Theme，我就不展开介绍。如果有机会，写点实际应用案例。如果基于案例介绍，会更有体悟吧。

还有，Fyne 的官方文档写的挺易读的，可直接看它的文档。

## 数据绑定

Fyne 从 v2.0.0 开始支持数据绑定。它让控件和与数据实时连接，数据更改会自动反映在UI上，反之亦然。

控件为了支持数据绑定能力，一般会提供如 `NewXXXWithData` 的接口。直接通过场景说明，场景：编辑输入框的内容，可直接实现到 Label 上。

核心代码如下所示：

```go
// 创建一个字符串绑定
textBind := binding.NewString()

// 创建一个 Entry，将其内容绑定到 textBind
entry := widget.NewEntryWithData(textBind)

// 创建一个 Label，也将其内容绑定到同一个 textBind
label := widget.NewLabelWithData(textBind)

// 使用容器放置 Entry 和 Label，以便它们都显示在窗口中
content := container.NewVBox(entry, label)
```

显示效果，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-13.gif)

尝试让前面自定义的 `CustomLabel` 支持数据绑定能力。只要创建一个 `NewCustomLabelWithData` 构造函数。

如下所示：

```go
func NewCustomLabelWithData(data binding.String) *CustomLabel {
	label := &CustomLabel{}
	label.ExtendBaseWidget(label)
	data.AddListener(binding.NewDataListener(func() {
		text, _ := data.Get()
		label.Text = text
		label.Refresh()
	}))
	return label
}
```

通过 `data` 监听数据变化后，立刻刷新组件。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-03/2024-03-01-fyne-in-golang-14.gif)

如上所示，我们这个自定义 Label 中的文本是居中显示的。

数据绑定的核心是监听器模式（Observer pattern）。每个绑定对象内部维护了一个监听器列表，当数据变化时，这些监听器会被通知更新。

在 Fyne 中，通过 `data.AddListner()` 将 UI 组件与数据绑定对象绑定时，实际上是在数据对象上注册了一个监听器，这个监听器会在数据变化时更新 UI 组件的状态。

## 结语

Fyne 是简单、强大和跨平台的 GUI 工具，使得用 GO 开发现代 GUI 应用多了一个优秀选择。随着对 Fyne 的深入，它能够更加灵活地构建出符合需求的应用。

我喜欢用 Go 的原因，很重要的原因就是它的简洁性，很容易看到本质的东西，但又无需理解太复杂的编程概念。
