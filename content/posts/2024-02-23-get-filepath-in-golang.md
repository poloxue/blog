---
title: "解决 Go 中文件路径的相关问题"
date: 2024-01-29T19:18:56+08:00
draft: true
comment: true
description: ""
---

- 配置和资源文件的路径；
- 执行命令路径；
  - exec.LookPath
    - 不需要 exec.LookPath，直接 filepath.Abs(os.Argv[0])
  - os.Executable
    - SymbolLink
  - 性能
- 所在目录路径；
  - os.GetCwd()
- 源码文件路径；
  - runtime.Caller(0)
- filepath.Dir()
  - 跨平台的路径问题；
- 定义配置和资源的规则；
  - 绝对路径规则；
  - 相对路径规则；
    - 确定根目录 - 即设置工作区根目录 pwd 当前目录；
  - 环境变量指定路径；
  - 使用嵌入资源；

获取文件路径是一个常见的需求。

### 方法一：使用`os.Executable`

从Go 1.8 开始，推荐的方法是使用 `os.Executable` 函数。

```go
ex, err := os.Executable()
if err != nil {
    panic(err)
}
exPath := filepath.Dir(ex)
fmt.Println(exPath)
```

这个方法返回启动当前进程的可执行文件的路径。使用`filepath.Dir`可以从这个路径中提取出目录。这种方法的优点是简单且跨平台，但它可能返回符号链接而不是实际路径

### 方法一（续）：使用`os.Executable`的优缺点

- **优点**：
  - **简单直接**：直接提供了启动当前进程的可执行文件的路径。
  - **跨平台兼容**：在不同的操作系统上都能正常工作。

- **缺点**：
  - **可能返回符号链接**：在某些情况下，可能返回用于启动进程的符号链接，而不是实际的可执行文件路径。
  - **不一定反映源文件位置**：如果你需要获取源代码文件的位置，这种方法可能不适用。

### 方法二：使用`runtime.Caller`和`filepath`包

另一种方法是结合使用`runtime.Caller`和`filepath`包。

```go
_, filename, _, _ := runtime.Caller(0)
dirname := filepath.Dir(filename)
```

这种方法通过`runtime.Caller`获取当前执行的文件的路径，然后使用`filepath.Dir`提取目录。

- **优点**：
  - **准确性**：提供了当前执行文件的确切位置，对于理解代码执行上下文非常有用。
  - **灵活性**：可以通过修改`runtime.Caller`的参数来获取不同调用栈层级的文件路径。

- **缺点**：
  - **复杂性**：相比`os.Executable`，这种方法更复杂，需要更多的理解和处理。
  - **可能与实际部署环境不符**：在某些部署环境中，编译后的程序可能不包含源代码路径信息。

### 方法三：使用第三方包`osext`

还有一种方法是使用第三方包`osext`，它提供了`ExecutableFolder`函数。

```go
folderPath, err := osext.ExecutableFolder()
if err != nil {
    log.Fatal(err)
}
fmt.Println(folderPath)
```

这个方法专门用于获取当前运行程序的可执行文件所在的目录。

- **优点**：
  - **专门设计**：专为获取可执行文件目录设计，通常很准确。
  - **跨平台**：支持多种操作系统。

- **缺点**：
  - **依赖第三方包**：需要引入外部依赖，可能会增加项目的复杂性。
  - **与`go run`不兼容**：在使用`go run`进行本地开发时可能不会如预期工作。

### 总结

选择哪种方法取决于具体的需求和环境。如果你需要一个简单且跨平台的解决方案，`os.Executable`可能是最佳选择。如果你需要更精确地控制或获取源代码文件的位置，`runtime.Caller`可能更合适。而如果你愿意引入第三方依赖以获得更专业的功能，`osext`包是一个不错的选择。
