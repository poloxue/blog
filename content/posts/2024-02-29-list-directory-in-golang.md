---
title: "2024 02 29 List Directory in Golang"
date: 2024-01-29T19:39:22+08:00
draft: true
comment: true
description: ""
---


在Go语言中，列出目录的内容是一个常见的操作。从Go 1.16版本开始，引入了一个新的函数`os.ReadDir`，它提供了一个简洁且高效的方式来读取目录中的文件和子目录。

### 使用`os.ReadDir`函数

`os.ReadDir`函数读取指定目录并返回一个`DirEntry`切片，其中包含按文件名排序的目录项。这个函数是乐观的，即使在读取目录项时发生错误，它也会尝试返回错误之前的文件名切片。

#### 示例代码

```go
package main

import (
    "fmt"
    "log"
    "os"
)

func main() {
    files, err := os.ReadDir(".")
    if err != nil {
        log.Fatal(err)
    }

    for _, file := range files {
        fmt.Println(file.Name())
    }
}
```

这段代码展示了如何使用`os.ReadDir`函数来列出当前目录中的所有文件和子目录的名称。

#### 优点

- **简洁性**：直接使用`os.ReadDir`，无需先打开目录。
- **效率**：返回的是`DirEntry`对象，比旧的`ioutil.ReadDir`返回的`FileInfo`对象更轻量。
- **错误处理**：在遇到错误时，它会尽可能返回已读取的目录项。

#### 缺点

- **兼容性**：只在Go 1.16及更高版本中可用。

### `fs.FileInfoToDirEntry`函数

Go 1.17引入了`fs.FileInfoToDirEntry`函数，它可以将`FileInfo`对象转换为`DirEntry`对象。这对于需要将旧代码迁移到使用`DirEntry`的情况很有用。

#### 示例用法

```go
info, _ := os.Stat("somefile")
dirEntry := fs.FileInfoToDirEntry(info)
```

这个函数主要用于与旧代码的兼容性和迁移，它允许你从`FileInfo`对象获取`DirEntry`对象。

#### 优点

- **兼容性**：有助于从旧代码迁移到新的API。
- **灵活性**：可以从现有的`FileInfo`对象获取`DirEntry`。

#### 缺点

- **间接性**：需要先获取`FileInfo`对象，再转换为`DirEntry`。

除了使用`os.ReadDir`和`fs.FileInfoToDirEntry`之外，还有其他几种方法可以用来列出目录中的文件和子目录。让我们来看看这些方法及其各自的优缺点。

###  使用`ioutil.ReadDir`函数（Go 1.16之前的版本）

在Go 1.16之前，`ioutil.ReadDir`是读取目录内容的常用方法。

#### 示例代码

```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"
)

func main() {
    files, err := ioutil.ReadDir(".")
    if err != nil {
        log.Fatal(err)
    }

    for _, f := range files {
        fmt.Println(f.Name())
    }
}

```

#### 优点

- **广泛使用**：在Go 1.16之前的版本中，这是标准的做法。
- **简单直接**：直接返回目录中所有文件的`FileInfo`列表。

#### 缺点

- **性能**：对于包含大量文件的目录，可能不如`os.ReadDir`高效。
- **已弃用**：在Go 1.16及更高版本中，`ioutil`包已被弃用。

### 使用`os.File`的`Readdir`方法

可以使用`os.Open`打开一个目录，然后调用`Readdir`方法来读取目录内容。

#### 示例代码

```go
package main

import (
    "fmt"
    "log"
    "os"
)

func main() {
    dir, err := os.Open(".")
    if err != nil {
        log.Fatal(err)
    }
    defer dir.Close()

    files, err := dir.Readdir(-1)
    if err != nil {
        log.Fatal(err)
    }

    for _, file := range files {
        fmt.Println(file.Name())
    }
}
```

#### 优点

- **灵活性**：可以指定读取的文件数量。
- **兼容性**：在所有Go版本中都可用。

#### 缺点

- **更多步骤**：需要先打开目录，然后读取，最后关闭目录。

### 使用`filepath.Walk`

`filepath.Walk`函数可以递归地遍历目录及其子目录。

#### 示例代码

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"
)

func main() {
    err := filepath.Walk(".", func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        fmt.Println(path)
        return nil
    })
    if err != nil {
        fmt.Printf("error walking the path %v: %v\n", ".", err)
    }
}
```

#### 优点

- **递归遍历**：可以遍历子目录。
- **灵活性**：可以在遍历过程中处理每个文件。

#### 缺点

- **性能**：对于大型目录或深层次的目录结构，可能比直接读取目录更慢。

每种方法都有其适用场景和优缺点。选择哪种方法取决于你的具体需求，比如你是否需要递归遍历子目录，你的Go版本，以及对性能的考虑。
