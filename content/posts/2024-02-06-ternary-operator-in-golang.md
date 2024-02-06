---
title: "Go 是否有三元运算符？Rust 和 Python 是怎么做的？"
date: 2024-02-06T08:00:00+08:00
draft: false
comment: true
description: "什么是三元运算符？在其他一些编程语言中，如 C 语言，三元运算符是一种可以用一行代码实现条件选择的简便方法。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-06-ternary-operator-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 技巧第十四篇，系列文章查看：[Go 语言技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

今天来聊聊在 Go 语言中是否支持三元运算符。这个问题很简单，没有。

首先，什么是三元运算符？

在其他一些编程语言中，如 C 语言，三元运算符是一种可以用一行代码实现条件选择的简便方法。

```c
x = condition ? a : b; // condition = true 则 x = a，否则 x = b
```

大道至简的 Go 中肯定是没有这个运算符。

今天这篇文章将会就此展开，介绍 Go 中三元运算符的一些实践。

让我们正式开始吧。

## 使用 `if-else` 语句

三元运算符，本质上其实就是 `if-else` 的简化版本。通过 `if-else` 实现自然就是最常用的做法。

```go
var x int
if condition {
    x = a
} else {
    x = b
}
```

非常简单且易理解，无心智负担。毕竟，这就应该是它本来的样子。

虽然这比三元运算符要长一些，但它更容易理解，也是 Go 所推荐的方式。

## 一行表达式

三元运算符之所以被人喜爱，我觉得重要的一个原因就是：它足够简洁。我们只要一行代码就实现条件判断。

在 Go 中，如果想在一行代码实现，可能吗？

我们先来看看 rust 和 Python 是如何实现的。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-06-ternary-operator-in-golang-02.png)

如果了解 rust，你可能看过如下代码。

```rust
let x = {
  if condition {
    a
  } else {
    b
  }
};
```


如上的代码中，我们创建了一个代码块，它的最后一个表达式会作为 `x` 的值。这是 rust 所支持的语法。其实现代的不少语言支持这种简约语法。

或者更简洁下写法也可以，如下：

```rust
let = if condition {a} else {b}
```

如果你了解 Python，你可能看到这样的代码。

```python
x = a if condition else b
```

是不是更加简洁。

Go 不支持这样的语法，我们要实现类似效果，就只能通过立刻执行的匿名函数实现。

代码如下：

```go
x := func() int {
  if condition {
     return a
  }
  return b
}()
```

算了，好丑，太麻烦了！

看起来还是 `if-else` 好用。但我还是不甘心，还是希望实现一行代码的效果，怎么办呢？

## If 函数

前面的示例中，我们通过匿名函数实现类似于三元运算符的功能。那不是说，我们预实现一个函数即可？

让我们写一个 `If` 的函数来模拟三元运算符。这个函数接收一个布尔值和两个可能的返回值。根据布尔值的真假，它返回其中一个值。

代码如下所示：

```go
func If(condition bool, a, b int) int {
    if condition {
        return a
    }
    return b
}

x := If(3 > 2, x1, x2)
```

现在的代码是不是就清晰了许多呢？

但这种方法还是有个缺点，就是针对不同的类型都要实现一个 `If` 函数，如 `IfInt()`、`IfString()`、`IfFloat()` 等等。

不过从 Go 1.18 开始，Go 成功引入泛型。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-06-ternary-operator-in-golang-03.png)

我们可以通过泛型扩展一个更通用的 If 函数，不仅仅适用于整数，还可以用于其他类型。

示例代码如下：

```go
func If[T any](condition bool, a, b T) T {
    if condition {
        return a
    }
    return b
}

func main() {
    x := 10
    result := If(x > 0, "positive", "negative")
    fmt.Println(result) // 输出 "positive"
}
```

当然，我也不是建议这么用。既然官方不支持就算了吧，`if-else` 多写几行就多写几行吧。

## 奇淫巧技：基于 `map` 

在网上，我还发现了一个奇淫巧技：基于 Map 模拟三元运算法。

代码如下：

```go
x = map[string]int{
  true: b,
  false: c,
}[a]
```

基于 `true` 和 `false` 实现条件判断。

这方法看起来挺有创意，但这其实会增加代码的理解成本，降低可读性。再者，这种方法的效率是没有 `if-else` 的效率高的，因为涉及到了 map 的算法实现，没有那么直接。

## 为什么 Go 没有三元运算符

你是否好奇，为什么 Go 语言没有三元运算符？

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-06-ternary-operator-in-golang-04.png)

官方认为三元运算符有时会让代码变得复杂和难以理解。Go 鼓励写出更清晰直接的代码。

一个 C 语言版本的复杂三元运算符示例代码：

```c
#include <stdio.h>

int main() {
    int x = 5, y = 10, z = 15;
    char *result;

    result = x > y ? "X" : 
             y > z ? "Y" : 
             z > x ? "Z" : 
             x == y ? "X equals Y" : 
             y == z ? "Y equals Z" : 
             x == z ? "X equals Z" : 
             "All equal";

    printf("%s\n", result);
    return 0;
}
```

看这个代码，头晕没？

我们看看摘自官方文档的原文：

> The reason ?: is absent from Go is that the language's designers had seen the operation used too often to create impenetrably complex expressions. The if-else form, although longer, is unquestionably clearer. A language needs only one conditional control flow construct.

翻译内容：

> Go 语言中没有 ?: 运算符的原因是，该语言的设计者们观察到这种运算符过于频繁地被用来创建难以理解的复杂表达式。尽管 if-else 形式更长，但它无疑更清晰。一种语言只需要一种条件控制流构造。

从 rust 和 python 的决策上也可看出，这个观点得到了很多人的认同。但与 Go 不同的是，rust 和 python 虽然不支持传统的三元运算符，它们都提供了其他简洁的写法。

不禁思考：Go 强调大道至简。但 rust 和 python 其实也挺简单的，依旧保留了三运算法符的优点。

## 总结

本文主要就 Go 中三元运算符展开讨论，从简单 `if-else` 语句、到基于匿名函数的单行表达式、及泛型抽象 If 函数等方式来实现类似的功能。当然，我没有建议使用这些方式，在没有内置支持的情况下，`if-else` 的写法就挺好的。

博文地址：[Go 中如何实现三元运算符？Rust 和 Python 是怎么做的？](https://www.poloxue.com/posts/2024-02-06-ternary-operator-in-golan/)
