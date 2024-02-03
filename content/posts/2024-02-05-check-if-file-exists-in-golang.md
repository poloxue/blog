---
title: "Go 中如何检查文件是否存在？可能产生竞态条件？"
date: 2024-02-05T08:00:00+08:00
draft: false
comment: true
description: "Go 语言如何检查文件是否存在呢？如果你用的是 Python，可通过 `os.path.exists` 这样的标准库函数实现。遗憾的是，Go 标准库没有提供这样直接的函数，但好在，没有直接的，却有不那么直接的方法。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 技巧第十三篇，系列文章查看：[Go 语言技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

Go 中如何检查文件是否存在呢？

如果你用的是 Python，可通过标准库中 `os.path.exists` 函数实现。遗憾的是，Go 标准库没有提供这样直接的函数，好在，没有直接的，却有不那么直接的。

本文将基于这个话题展开，介绍 Go 中如何检查文件是否存在。

另外，本文最后还会介绍一个小注意点，即在判断文件是否存在时，如何避免中潜在的竞态条件。

## `os.Stat` 检查文件状态

Go 标准库虽然没有提供类似于 `os.Exist` 这样直接的函数检查文件是否存在，但它提供另外一个函数 `os.Stat`。

`os.Stat` 函数的作用是获取文件状态信息，我们通过检查它返回的错误即可知晓文件是否存在。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-02.png)

示例代码，如下所示：

```go
func main() {
  _, err := os.Stat("/path/to/file")
  if err != nil {
    if os.IsNotExist(err) {
      // 文件不存在
    } else {
      // 其他错误
    }
  }
  // 文件存在
}
```

第一个返回值表示文件信息，不是我们关心的重点，直接省略掉。

第二个返回值表示错误 `error`。如果文件不存在，可通过检查 `os.IsNotExist` 检查 `error` 是否是 `os.ErrNotExist`，确定文件是否存在。

## 与 C 对比

上面的示例中，我们使用 `os.Stat` 函数获取文件的状态，通过 `errors.Is` 判断返回错误，如果是 `os.ErrNotExist`，则文件不存在。

不得不说，这其实更底层更标准的做法。

类似于 Python 等高级语言，提供 `os.path.exist` 主要是为了方便编程，提高效率。

如果使用 Unix C 实现同样的功能，示例代码如下：

```c
#include <errno.h>
#include <stdio.h>
#include <sys/stat.h>

int main() {
  struct stat buffer;
  int exist = stat("/path/to/file", &buffer);
  if (exist != 0) {
    if (errno == ENOENT) { /* 文件不存在*/ } 
    else { /* 其他错误 */ }
    return 0;
  }
  // 文件存在
  return 0;
}
```

是不是和我们前面代码基本是一个模子。

## Go1.13 以及之后推荐使用 `errors.Is`

自 Go 1.13 起，推荐使用 `os.Stat` 和 `errors.Is` 的组合。这种方法提供了更一致和灵活的错误处理方式。

具体而言，即使是经过包裹的错误，`errors.Is` 依然能够识别。

我期初认为，`os.IsNotExist` 能识别包裹 `error`，但不太确定，于是写了个代码简单测试了下。

示例代码，如下所示：

```go
_, err := os.Stat("/path/to/file")  // 这是一个不存在的文件路径
werr := fmt.Errorf("Main: %w", err) // 包裹生成新错误
fmt.Println(os.IsNotExist(err))     // 返回 true，表示不存在，这是错误结果
fmt.Println(os.IsNotExist(werr))    // 返回 false，表示存在
fmt.Println(errors.Is(werr, os.ErrNotExist)) // 返回 true 表示不存在
```

测试结果都已写在注释中。

如上可知， `os.IsNotExist` 只能识别最初的 `error`，如果错误经过 `fmt.Errorf` 包裹，则必须使用 `errors.Is` 识别。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-03.png)

一句话概括，`os.IsNotExist` 可以用，但有适用范围，而 `errors.Is` 则更通用。

这一般也同样适用于其他类似的库。

## 直接使用 Open 避免竞态条件

到这里，基本已经解答了 Go 中如何检查文件存在性的问题。

但，我还想引入一个讨论：并发场景下，如何避免检查文件存在性时引入潜在的竞态条件？

简言之，文件状态可能在检查和操作发生变化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-04.png)

什么是更好的做法呢？

我们可以直接尝试打开或操作文件，根据返回结果判断错误。

示例代码如下：

```go
file, err := os.Open("/path/to/file")
if err != nil {
    if errors.Is(err, os.ErrNotExist) {
        // 文件不存在
    } else {
        // 处理其他类型的错误
    }
}
```

如上代码中，你通过 `open` 直接打开一个文件，如果文件不存在，`os.Open` 将返回一个错误，我们检查 error 确定下一步的操作。

通过这种方式，我们可以避免打开文件时引入竞态条件。

## open 是原子操作？

读到这里，可能有人不禁问，为什么 open 能避免竞态条件呢？它是原子操作吗？

是的。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-05-v1.png)

系统调用都是原子操作，操作系统会保证操作过程不受到干扰。如果出现问题，也会进行回滚操作.

这一点对于 Open 同样使用。

当我们使用 open 打开一个文件时，系统会确保在这个操作完成前，不会受其他操作干扰，包括如检查文件是否存在、创建文件描述符、分配必要的资源等。

## 结论

本文通过一个小小的问题：Go 语言中如何检查文件是否存在，除了引出 Go 中检查文件是否存在的基本方法。同时，还介绍了文件操作时如何避免潜在的竞态条件，进一步了解到一个有趣的小知识，Unix 系统调用是原子性操作。

最后，还是希望本文能帮助各位在 GO 语言的学习道路上起到一点微末作用。

博客地址：[Go 中如何检查文件是否存在？可能产生竞态条件？](https://www.poloxue.com/posts/2024-02-05-check-if-file-exists-in-golang/)

