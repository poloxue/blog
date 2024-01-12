---
title: "如何理解 Go 的接口"
date: 2019-06-18T16:09:33+08:00
draft: false
comment: true
tags: ["Golang"]
---

[如何理解 Golang 中的接口](https://www.zhihu.com/question/318138275/answer/699989214)。

个人认为，要理解 Go 的接口，一定先了解下鸭子模型。

# 鸭子模型

那什么鸭子模型？

鸭子模型的解释，通常会用了一个非常有趣的例子，一个东西究竟是不是鸭子，取决于它的能力。游泳起来像鸭子、叫起来也像鸭子，那么就可以是鸭子。

动态语言，比如 Python 和 Javascript 天然支持这种特性，不过相对于静态语言，动态语言的类型缺乏了必要的类型检查。

Go 接口设计和鸭子模型有密切关系，但又和动态语言的鸭子模型有所区别，在编译时，即可实现必要的类型检查。

# 什么是 Go 接口

Go 接口是一组方法的集合，可以理解为抽象的类型。它提供了一种非侵入式的接口。任何类型，只要实现了该接口中方法集，那么就属于这个类型。

举个例子，假设定义一个鸭子的接口。如下：
```go
type Duck interface {
	Quack()   // 鸭子叫
	DuckGo()  // 鸭子走
}
```

假设现在有一个鸡类型，结构如下：
```go
type Chicken struct {
}

func (c Chicken) IsChicken() bool {
	fmt.Println("我是小鸡")
}
```

这只鸡和一般的小鸡不一样，它比较聪明，也可以做鸭子能做的事情。
```go
func (c Chicken) Quack() {
	fmt.Println("嘎嘎")
}

func (c Chicken) DuckGo() {
	fmt.Println("大摇大摆的走")
}
```

注意，这里只是实现了 Duck 接口方法，并没有将鸡类型和鸭子接口显式绑定。这是一种非侵入式的设计。

我们定义一个函数，负责执行鸭子能做的事情。

```go
func DoDuck(d Duck) {
	d.Quack()
	d.DuckGo()
}
```
因为小鸡实现了鸭子的所有方法，所以小鸡也是鸭。那么在 main 函数中就可以这么写了。
```
func main() {
	c := Chicken{}
	DoDuck(c)
}
```

执行正常。如此是不是很类似于其他语言的多态，其实这就是 Go 多态的实现方法。

# 空接口

继续说说空 interface。

如果一个 interface 中如果没有定义任何方法，即为空 interface，表示为 interface{}。如此一来，任何类型就都能满足它，这也是为什么当函数参数类型为 interface{} 时，可以给它传任意类型的参数。

示例代码，如下：

```go
package main

import "fmt"

func main() {
	var i interface{} = 2
	fmt.Println(i)
}
```

更常用的场景，Go 的 interface{} 常常会被作为函数的参数传递，用以帮助我们实现其他语言中的泛型效果。Go 中暂时不支持 泛型，不过 Go 2 的方案中似乎将支持泛型。

# 总结

回答结束，做个简单总结。理解 Go 接口要记住一点，接口是一组方法的集合，这句话非常重要，理解了这句话，再去理解 Go 的其他知识，比如类型、多态、空接口、反射、类型检查与断言等就会容易很多。

我的博文：[如何理解 Go 的接口](https://www.poloxue.com/posts/2019-06-18-understand-golang-interface)
