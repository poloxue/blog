---
title: "Go 中如何实现三元运算符？"
date: 2024-01-28T22:48:33+08:00
draft: true
comment: true
description: ""
---

今天来聊聊在Go语言中如何实现类似三元运算符的功能。

首先，什么是三元运算符？

在一些其他的编程语言中，比如C语言，三元运算符是一种可以在一行代码内根据条件选择不同结果的简便方法。

但 Go 语言中，我们没有这个运算符。不过别担心，我会告诉你我们怎么用其他方法来达到同样的目的。

## 使用 `if-else` 语句

在Go语言中，最常用的方法是`if-else`语句。这就像是在说：“如果这个条件是真的，我就做这件事；否则，我就做另一件事。”虽然这比三元运算符要长一些，但它更容易理解，也是Go语言推荐的方式。

比如，我们有一个叫`val`的变量，我们想根据它的值是正还是负来赋值给`index`变量。我们可以这样写：

```go
var index int
if val > 0 {
    index = val
} else {
    index = -val
}
```

## 使用立即执行的匿名函数

如果你需要在一行代码内完成这个操作，我们可以使用一个特殊的方法，叫做立即执行的匿名函数。这就像是我们创建了一个没有名字的小函数，并且马上使用它。这样做可以确保只有一个条件被执行。

看看这个例子：

```go
index := func() int {
    if val > 0 {
        return val
    }
    return -val
}()
```

### If 函数

我们还可以自己写一个叫`If`的函数来模拟三元运算符。这个函数接收一个布尔值和两个可能的返回值。根据布尔值的真假，它返回其中一个值。

就像这样：

```go
func If(bool_value bool, value1, value2 int) int {
    if bool_value {
        return value1
    }
    return value2
}

index := If(val > 0, val, -val)
```

## 奇淫巧技：基于 `map` 

有些人提出了使用`map`的方法来模拟三元运算符。这是一种很有创意的方法，但可能会让代码变得难以理解，而且可能影响程序的运行效率。

## Go 1.18 泛型支持

在Go 1.18版本中，引入了泛型的支持，这意味着我们可以创建一个可以处理不同类型的If函数。使用泛型，我们可以写一个更通用的If函数，它不仅仅适用于整数，还可以用于其他类型，比如字符串、浮点数等。

让我们来看一个使用泛型的If函数的例子：

```go
package main

import "fmt"

func If[T any](condition bool, trueVal, falseVal T) T {
    if condition {
        return trueVal
    }
    return falseVal
}

func main() {
    val := 10
    result := If(val > 0, "positive", "negative")
    fmt.Println(result) // 输出 "positive"
}
```

## 为什么 Go 没有三元运算符

你可能会好奇，为什么Go语言没有三元运算符呢？原因是Go的设计者们认为三元运算符有时会让代码变得复杂和难以理解。Go语言鼓励写出更清晰、更直接的代码。

## 总结

虽然Go没有内置的三元运算符，但我们可以通过`if-else`语句或立即执行的匿名函数等方式来实现类似的功能。这些方法都很有用，可以根据你的需要和你觉得哪种方法更容易理解来选择使用。

