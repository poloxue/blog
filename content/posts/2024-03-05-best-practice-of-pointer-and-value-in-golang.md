---
title: "2024 02 20 Best Practice of Pointer and Value in Golang"
date: 2024-02-29T18:36:18+08:00
draft: true
comment: true
description: ""
---

### Go中指针和值作为函数参数和返回值的最佳实践

在Go语言中，理解何时使用指针和何时使用值对于编写高效和可维护的程序至关重要。本文将为初学者提供一个从简单到复杂的指南，帮助你理解在不同情况下应该如何选择。

#### 方法接收者

当我们谈论方法的接收者时，一个常见的建议是：“如果有疑问，使用指针。” 这是因为方法经常需要修改它们所作用的对象。例如，如果你有一个大型结构体，使用指针可以避免复制的开销。

```go
type MyStruct struct {
    Val int
}

func (s *MyStruct) SetValue(val int) {
    s.Val = val
}
```

在这个例子中，`SetValue`方法使用指针接收者，这样它就可以修改`MyStruct`实例的`Val`字段。

#### 内置类型

对于内置类型，如切片（slices）、映射（maps）、通道（channels）、字符串（strings）、函数值（function values）和接口值（interface values），它们内部已经实现了指针，通常不需要再次使用指针。

#### 使用指针的情况

- 如果函数需要修改接收者，则接收者必须是指针。
- 如果接收者是大型结构体，使用指针接收者更高效。
- 为了一致性，如果类型的某些方法必须有指针接收者，那么其他方法也应该使用指针接收者。

#### 使用值的情况

- 对于小型结构体，特别是那些不需要修改的结构体，使用值传递可以避免内存分配和垃圾回收的开销。
- 值传递可以避免意外的别名情况，即在一个地方的赋值操作意外地改变了另一个地方的值。
- 对于小型结构体，值传递可能更高效，因为它避免了缓存未命中或堆分配。

```go
type SmallStruct struct {
    Val int
}

func PassByValue(s SmallStruct) {
    s.Val = 10
}
```

在这个例子中，`PassByValue`函数通过值接收`SmallStruct`实例，这意味着它接收的是一个副本，对它的修改不会影响原始实例。

#### 切片和映射的特殊情况

- 对于切片，你不需要传递指针来改变数组的元素。例如，`io.Reader.Read(p []byte)`可以改变`p`的字节。
- 对于需要重新切片的切片（即改变切片的开始位置、长度或容量），内置的函数如`append`接受一个切片值并返回一个新的切片。

```go
func AppendSlice(slice []int, value int) []int {
    return append(slice, value)
}
```

在这个例子中，`AppendSlice`函数接受一个切片并返回一个新的切片，这个新切片包含了添加的元素。

通过这些示例和解释，我们可以看到，在Go语言中，根据不同的情况和需求选择使用指针还是值是非常重要的。理解这些概念将帮助你编写更高效、更可维护的Go程序。
