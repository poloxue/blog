---
title: "推荐一个可用于快速创建 TUI 应用的框架 - Bubble Tea"
date: 2024-02-19T08:00:00+08:00
draft: false
comment: true
description: "今天介绍一个 TUI 库 - Bubble Tea，一个小巧但强大的文本用户界面（TUI）框架，基于 Go 语言开发。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-01.png)

嗨！大家好，我是波罗学。本文是 Golang 三方库推荐第二篇，系列查看：[Golang 三方库](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MzE2NTY2MA==&action=getalbum&album_id=3302384940181110785#wechat_redirect)。

今天介绍一个 TUI 库 - [Bubble Tea](https://github.com/charmbracelet/bubbletea)，一个小巧但强大的文本用户界面（TUI）框架，基于 Go 语言开发。

## 什么是 TUI？

什么是 TUI？为了防止可能有人不知道，简单解释一下。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-02.gif)

TUI，全称为文本用户界面（Text-based User Interface），是一种用户界面类型。对之对比的就是 GUI（Graphical User Interface）。我开始还以为 TUI 中的 T 的全称是 terminal。

TUI 是通过纯文本和基本字符在终端或控制台窗口中展示信息和交互选项。与 GUI 不同，TUI 不依赖图形元素如窗口、图标或鼠标指针，而是使用文本字符来构建用户界面的布局和元素。

如同我们开发 GUI 应用需要 GUI 框架的帮助，开发 TUI 应用也需要 TUI 框架的帮助，而 Bubble Tea 就是这样一款 TUI 框架。

## 使用 Bubble Tea

Bubble Tea 受到 The Elm Architecture 的启发，基于这种有趣、功能性强且保持状态的架构来构建终端应用程序。无论是简单的行内应用程序、全窗口应用程序，还是两者的混合，Bubble Tea 都可使用。

要学会基于 Bubble Tea 开发终端应用，核心要理解它的三个基本部分组成：模型（Model）、更新函数（Update）和视图函数（View）。而 `Update` 和 `View` 都是属于 Model 的方法。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-05.png)

简单解释下它们的作用，如下:

- **模型（Model）**：描述应用程序状态的数据结构。
- **更新函数（Update）**：处理输入事件（如按键、计时器等），并更新模型。
- **视图函数（View）**：根据模型的当前状态渲染用户界面。

不明白？我们来通过一个例子解释下。

## 一个简单的例子：TUI 的计数器应用。

这是一个计数器的案例，我们通过输入 '+' 号增加计数值，减号减少计数值，输入 'q' 退出应用。

计数器的效果图，如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-03.gif)

如何基于 Bubble Tea 实现这个小应用呢？我们只要实现一个简单的 Model 即可。

如下是它最核心的 `model` 代码。

```go
type model int

// 初始化模型，设置计数器的初始值
func initialModel() model {
	return 0
}

func (m model) Init() tea.Cmd {
	return nil
}

// 更新函数，根据消息更新模型
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		case "q":
			return m, tea.Quit // 按下 'q' 退出程序
		case "+":
			return m + 1, nil // 按下 '+' 增加计数
		case "-":
			return m - 1, nil // 按下 '-' 减少计数
		}
	}
	return m, nil
}

// 视图函数，渲染界面
func (m model) View() string {
	return fmt.Sprintf("Count: %d\nPress + to increment, - to decrement, q to quit.\n", m)
}
```

观察以上代码，model 是一个基于 int 创建的新类型，用于保存应用的状态信息，即计数值。initialModel 创建一个初始值为 0 的 model。model 还提供了两个核心方法，分别是 `Update` 和 `View`。

如上段所述，`Update` 是用于接收用户消息，更新 model 状态。`View` 函数用于更新界面内容，展示给用户，它的更新通过 `Printf` 打印即可。

现在，只需将 `Model` 实例交给 Bubble Tea 框架运行皆可。

```go
func main() {
	p := tea.NewProgram(initialModel())
	if _, err := p.Run(); err != nil {
		fmt.Fprintf(os.Stderr, "Error running program: %v", err)
		os.Exit(1)
	}
}
```

这个应用到此就开发完成。

## 核心流程

通过 Counter 计数器这个简单案例，重新理解下 Bubble Tea 中 The Elm Architecture 数据流向。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-04.png)

1. 用户与界面交互，触发消息。
2. 消息被发送到 Update 函数，Update 函数根据消息更新 Model。
3. 更新后的 Model 被传递给 View 函数，View 函数生成新的界面显示给用户。

现在，理解起来是不是非常简单了。

## 一个小游戏：贪吃蛇

基于 Bubble Tea 框架，开发了一个非常简单粗糙的贪吃蛇游戏，不会增加蛇的长度，效果如下：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-12.gif)

S 表示 `snake`，F 表示 Food，通过 w -> up, s -> donw, a -> left, d -> right 控制方向。

代码地址：[贪吃蛇游戏](https://gist.github.com/poloxue/f5478b535f9bf507bc3cc408332c2090)。有兴趣可自行扩展。

## 更多示例

我们通过一个简单的计数案例解释了 Bubble Tea 的使用。如果想了解它的更多能力，可参考 Bubble Tea 源码中的 [examples](https://github.com/charmbracelet/bubbletea/tree/master/examples/) 了解它更多的能力。

来自仓库中的一些案例效果图：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-06.gif)
![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-07.gif)
![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-08.gif)
![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-09.gif)

更多查看：[bubble tea's examples](https://github.com/charmbracelet/bubbletea/tree/master/examples)。

此外，仓库中还提供了[教程](https://github.com/charmbracelet/bubbletea/tree/master/tutorials/)，帮助我们理解如何使用这个框架构建终端应用程序。

## 其他 TUI 框架

相对而言，bubble tea 提供构建终端应用的基本框架，但如果你希望开发非常复杂应用，它确实还是有一些拙荆见肘，如常用的窗口组件或基本的布局能力，它是没有提供的。

这里再推荐 Go 中其他一些可供选择的 TUI 框架。

首先是 tview，一个基于 tcell 的富文本终端应用程序框架，提供了构建复杂交互 TUI 所需的组件和布局管理器。它支持颜色、表格、表单、列表等多种控件，非常适合构建复杂的 TUI 应用。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-10.gif)

其次是 termui:，一个基于 Go 的 TUI 库，灵感来源于 blessed-contrib 和 tui-rs。它提供了创建数据可视化和仪表板的简单方法，支持条形图、线图、饼图等多种图表类型，适用于构建需要数据展示的 TUI 应用。不过这个库已经很久没有更新了，如果你熟悉 rust，推荐使用 tui-rs。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-19-tui-library-bubble-tea-in-golang-11.gif)

## 总结

本文主要推荐一个 Golang 的 TUI 开发库 - [Bubble Tea](https://github.com/charmbracelet/bubbletea)，如果你对这个话题感兴趣，或者有相关需求，可考虑一试。

博文地址：[推荐一个可用于快速创建 TUI 应用的框架 - Bubble Tea](https://www.poloxue.com/posts/2024-02-19-tui-library-bubble-tea-in-golang/)

