---
title: "一文理清 Go 引用的常见疑惑"
date: 2019-09-28T14:40:56+08:00
draft: false
comment: true
tags: ["Golang"]
---

今天，尝试谈下 Go 中的引用。

之所以要谈它，一方面是之前的我也有些概念混乱，想梳理下，另一方面是因为很多人对引用都有疑问。我经常会看到与引用有关的问题。

比如，什么是引用？引用和指针有什么区别？Go 中有引用类型吗？什么是值传递？址传递？引用传递？

在开始谈论之前，我已经感觉到这必定是一个非常头疼的话题。这或许就是学了那么多语言，但没有深入总结，从而导致的思维混乱。

# 前言

我的理解是，要彻底搞懂引用，得从类型和传递两个角度分别进行思考。

从类型角度，类型可分为值类型和引用类型，一般而言，我们说到引用，强调的都是类型。

从传递角度，有值传递、址传递和引用传递，传递是在函数调用时才会提到的概念，用于表明实参与形参的关系。

引用类型和引用传递的关系，我尝试用一句话概括，引用类型不一定是引用传递，但引用传递的一定是引用类型。

这几句话，是我在使用各种语言的之后总结出来的，希望无误吧，毕竟不能误导他人。

# 是什么

谈到引用，就不得不提指针，而指针与引用是编程学习中老生常谈的话题了。有些编程语言为了降低程序员的使用门槛，只有引用。而有些语言则是指针引用皆存在，如 C++ 和 Go。

指针，即地址的意思。

在程序运行的时候，操作系统会为每个变量分配一块内存放变量内容，而这块内存有一个编号，即内存地址，也就是变量的地址。现在 CPU 一般都是 64 位，因而，这个地址的长度一般也就是 8 个字节。

引用，某块内存的别名。

一般情况，都会这么解释引用。换句话说，引用代指某个内存地址，这句话真的是非常简洁，同时也非常好理解。但在 Go 中，这句话看起来并不全面，具体后面解释。

除了指针和引用，还有另外一个更广泛的概念，值。谈变量传递时，常会提到值传递、址传递和引用传递。从广义上看，对大部分的语言而言，指针和引用都属于值。而从狭义角度来说，则可分为值、址和引用。

相当绕人是不是？

我已经感觉到自己头发在掉了。其实，要想彻底搞清楚这些概念，还是得从本质出发。

# 值和指针

先来搞明白值与指针区别。

上一节在介绍指针的时候，提到了要注意变量的地址和内容的不同。为什么要说这句话呢？

假设，我们定义一个 int 类型的变量 a，如下：

```go
var a int = 1
```

变量 a 的内容为 1，而变量内容是存在某个地址之中的。如何获取变量地址呢？Go 中获取变量地址的方法与 C/C++ 相同。代码如下：

```go
var p = &a
```

通过 & 获取 a 的地址。同时，这里还定义了一个新的变量 p 用于保存变量 a 的地址。p 的类型为 int 指针，也就是变量 p 中的内容是变量 a 的地址。

如下代码输出它们的地址：

```go
var a = 1
var p = &a
fmt.Printf("%p\n", p)
fmt.Printf("%p\n", &p)
```

我这里的输出结果是，变量 a 和 p 的地址分别为 0xc000092000 和 0xc00008c010。此时的内存的分布如下：

![](https://blogimg.poloxue.com/0011-go-reference-01.png?v=11)

变量 p 的内容是 a 的地址，因而可以说指针即是其他变量的内容，也是某个变量的地址。为什么啰啰嗦嗦的说这些，因为在学习 C 语言，会单独强调址的概念，但在 Go 中，指针相对弱化，也是归于值类型之中。

# 引用的本质

前面说过，引用是某块内存的别名。从字面理解，似乎表达的是引用类型变量中的内容是指针，这么理解似乎也没错。既然如此，我自然而然地想到，怎么将引用与指针关联起来。


在 C/C++ 中，引用其实是编译器实现的一个语法糖，经过汇编后，将会把引用操作转化为了指针操作。这真的是别名啊，有种 define 预处理的感觉，只不过是汇编级别的。分享一篇 [C++中“引用”的底层实现](https://www.cnblogs.com/hoodlum1980/archive/2012/06/19/2554270.html) 的文章，有兴趣仔细读读，我只是看了个大概。

而其他一些语言中，引用的本质其实是 struct 中包含指针，比如 Python。下面的 C 结构是 Python 中列表类型的底层结构。

```c
typedef struct {
    PyObject_VAR_HEAD

    PyObject **ob_item;

    Py_ssize_t allocated;
} PyListObject;
```

变量真正存放数据的地方在 `**ob_item` 中。结构中的其他两个成员起辅助作用。

现在看来，引用的实现主要有两种。一是 C++ 的思路，引用其实一种便于使用指针的语法糖，和我们想象中的别名含义一致。二是类似 Python 中的实现，底层结构中包含指向实际内容的指针。

当然，或许还有其他的实现方式，但核心应该是不变的。

# 引用传递

谈到引用传递，就不得不提值传递，值传递的一般定义如下。

函数调用时，实参通过拷贝将自身内容传递给形参，形参实际上是实参值的一个拷贝，此时，针对函数中形参的任何操作，仅仅是针对实参的副本，不影响原始值的内容。

值传递中有一个特殊形式，如果传递参数的类型是指针，我们就会称之为址传递，C 语言中就有值传递和址传递两种说法。深究起来，C 中的址传递也属于值传递，因为对指针类型而言，变量的值是指针，即传递的值也是指针。而 C 语言之所以强调址传递，我认为主要 C 这门底层语言对指针较为重视。

什么是引用传递？

参考值传递的定义，实参地址在函数调用被传递给形参，针对形参的操作，影响到了实参，则可以认为是引用传递。

在我用过的语言中，支持引用传递的语言有 PHP 和 C++。

# Go 的引用实现

Go 的引用类型有 slice、map 和 chan，实现机制采用的是前面提到的第二种方式，即结构体含指针成员。它们都可以使用内置函数 make 进行初始化。

原本我是想把这几种引用类型的底层结构都贴出来，但发现这会干扰本文主题的理解。我们只看 slice 的结构，如下：

```go
// slice
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

slice 的结构最简单，包含三个成员，分别是切片的底层数组地址、切片长度和容量大小。是否感觉与前面提到的 Python 列表的底层结构非常类似？

如果想了解 map 和 chan 的结构，可自行阅读 go 的源码，[runtime/slice.go](https://github.com/golang/go/blob/master/src/runtime/slice.go)、[runtime/map.go](https://github.com/golang/go/blob/master/src/runtime/map.go) 和 [runtime/chan.go](https://github.com/golang/go/blob/master/src/runtime/chan.go)。

如果不想研究源码，推荐阅读饶大的 Go 深度解密系列文章，包括 [深度解密Go语言之Slice](https://mp.weixin.qq.com/s/MTZ0C9zYsNrb8wyIm2D8BA)、[深度解密Go语言之map](https://mp.weixin.qq.com/s/2CDpE5wfoiNXm1agMAq4wA)、[深度解密Go语言之channel](https://mp.weixin.qq.com/s/90Evbi5F5sA1IM5Ya5Tp8w)，这几篇文章因为写的都非常细且非常长，可能读起来会比较考验你的耐心。

# Go 是值传递

按官方说法，Go 中只有值传递。原文如下：

> In a function call, the function value and arguments are evaluated in the usual order. After they are evaluated, the parameters of the call are passed by value to the function and the called function begins execution. The return parameters of the function are passed by value back to the calling function when the function returns.

重点是下面这句话。

> After they are evaluated, the parameters of the call are passed by value to the function and the called function begins execution. 

有点迷糊？最初我也迷糊，Go 不是有指针和引用类型吗。但读了一些文章，思考了许久，才彻底想明白。下面，我将尝试为官方的说法找个合理的解释。

**为什么说 Go 中没有址传递**

其实，这个问题前面已经解释的很清楚了，指针只是值的一种特殊形式，C 语言是门非常底层的语言，常会涉及一些地址操作，会强调指针的特殊地位。但于 Go 而言，指针已经弱化了很多，Go 团队可能也觉得没有必要再单独强调指针的地位。

**为什么说 Go 中没有引用传递？**

有人可能会说，Go 中明明有引用传递，按照引用传递的定义，可以非常容易就拿出一个例子反驳我。

```go
package main

import "fmt"

func update(s []int) {
	s[1] = 10
}

func main() {
	a := []int{0, 1, 2, 3, 4}
	fmt.Println(a)
	update(a)
	fmt.Println(a)
}
```

输出结果如下：

```bash
[0 1 2 3 4]
[0 10 2 3 4]
```

针对形参 s 的操作确实改变了实参 a 的值，似乎的确是引用传递。但我想说的是，针对形参的操作并非指的是针对形参中某个元素的操作。

看个 C++ 中引用的例子。

```cpp
void update(int& s) {
	s = 10;
	printf("s address: %p\n", &s);
}

int main() {
	int a = 1;
	std::cout << a << std::endl;
	printf("a address: %p\n", &a);
	update(a);
	std::cout << a << std::endl;
}
```

执行结果如下：

```
1
a address: 0x7fff5b98f21c
s address: 0x7fff5b98f21c
10
```

针对 s 的操作确实改变了 a 的值。在 Go 中尝试同样的代码，如下：

```go
func update(s []int) {
	s[1] = 10
	fmt.Printf("%p\n", &s)
}

func main() {
	a := []int{0, 1, 2, 3, 4}
	fmt.Println(a)
	fmt.Printf("%p\n", &a)
	update(a)
	fmt.Println(a)
}
```

输出如下：

```go
[0 1 2 3 4]
0xc00000c060
0xc000098000
[0 10 2 3 4]
```

非常遗憾，针对形参的赋值操作并没有改变实参的值。基于此，得出结论是 slice 的传递并非引用传递。我比较喜欢的这种解释方式，适合我个人的记忆理解，不知道是否有不妥的地方。

除此之外，介绍另外一种识别是否是引用传递的方式。

通过比较形参和实参地址确认，如果两者地址相同，则是引用传递，不同则非引用传递。但因为 C++ 和 Go 引用的实现机制不同，理解起来会比较困难。我们也可以选择只记结论。

这种方式的验证非常简单，我们在上面的 C++ 和 Go 的例子中已经输出了形参和实参的地址，比较下即可得出结论。

# 总结

本文主要从引用的类型和传递两个角度出发，深入浅出的分析了 Go 中的引用。

首先，引用类型和引用传递并没有绝对的关系，不知道有多少人认为引用类型必然是引用传递。接着，我们讨论了不同语言引用的实现机制，涉及到 C++、Python 和 Go。

文章的最后，解释了一个常见的疑惑，为什么说 Go 只有值传递。在此基础上，文中提出了两种方式，帮助识别一门语言是否支持引用传递。

# 相关阅读

[golang中哪些引用类型的指针在声明时不用加&号，哪些在函数定义的形参和返回值类型中不用*号标注](https://segmentfault.com/q/1010000019965306/a-1020000019996800)

[Golang中的make(T, args)为什么返回T而不是*T?](https://www.zhihu.com/question/312356800/answer/739572672)  

[Go语言参数传递是传值还是传引用](https://www.flysnow.org/2018/02/24/golang-function-parameters-passed-by-value.html)

[Golang中函数传参存在引用传递吗？](https://juejin.im/post/5b1dcce0e51d4506a74d4364)

[C++ 引用 底层实现机制](https://blog.csdn.net/cFarmerReally/article/details/54580711)

[The Go Programming Language Specification](https://golang.org/ref/spec#Calls)


我的博文：[一文理清 Go 引用的常见疑惑](https://www.poloxue.com/posts/2019-09-28-understand-golang-reference)
