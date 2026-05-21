---
date: 2024-12-12
title: "主题 Icon 小图标"
weight: 6
---

以下每个图标都可以通过`theme`包作为一个函数获得。例如`theme.InfoIcon()`。

这些图标也可以通过使用`ThemeIconName`以及在实现了`fyne.Theme`的结构体上的`Icon`方法，通过它们的源图标名称获得。例如`theme.Icon(theme.IconNameInfo)`。

## 列表

Fyne 提供丰富的内置图标，完整列表请查看 [Fyne 官方图标文档](https://docs.fyne.io/api/v2.4/theme/)。

以下是常用图标的调用方式：

- `theme.InfoIcon()`
- `theme.AccountIcon()`
- `theme.ArrowDropDownIcon()`
- `theme.ArrowDropUpIcon()`
- 以及更多... 所有图标都可通过 `theme.Icon(theme.IconNameXxx)` 的方式引用。

## 使用其他颜色集

每个图标都可以作为特定主题颜色的源使用各种公共帮助方法：

* `NewDisabledThemedResource`
* `NewErrorThemedResource`
* `NewInvertedThemedResource`
* `NewPrimaryThemedResource`

默认情况下，所有图标都适应当前主题前景色，使用`NewThemedResource`，它使用主题前景色。所有图标都是SVG `width="24"`, `height="24"`。


