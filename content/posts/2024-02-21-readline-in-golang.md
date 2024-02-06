---
title: "基于 bufio 包的 Reader 和 Scanner 按行读取文件"
date: 2024-02-06T12:09:49+08:00
draft: true
comment: true
description: "在编程时，按行读取文件是一个很常规的需求，它相较于一次性读出整个文件，有着诸如内存效率高、处理速度快、实时性高、可扩展性强等优势。"
---

在编程时，按行读取文件是一个很常见的需求，它相较于一次性读出整个文件，有如下这些优势。

**内存效率高**，按行读取，减少了内存占用；

**处理速度快**，逐行处理可更快开始，而且还可利用并行最大化利用计算资源；

**实时性高**，实时流数据，如日志数据等，按行处理将会更加实时；

**可扩展性强**，按行读取不会可同时适用于小文文件和大文件的处理，统一的处理方式；

**灵活性高**，按需读数据和处理数据，只有终止了处理过程，就终止读取其他无用的数据；

在 Python 中，按行读取文件非常直观和简单。好吧，我最喜欢拿 Python 作为对比语言，毕竟它最易用。

示例代码，如下所示：

```python
with open('filename.txt', 'r') as file:
    line = file.readline()
    while line:
        # 处理每一行
        print(line)
        line = file.readline()
```

那么，Go 语言中，如何按行读取文件呢？

本文将基于这个主题展开，介绍 Go 如何按行读取文件。重点在于 `bufio.Reader` 和 `bufio.Scanner`  的使用。后面考虑再重点写一篇如何处理大文件的文章。

让我们开始吧。

## 准备一个文本文件

我们先准备一个短小的文本文件 example.txt，内容如下：

```plain
This post covers the Golang Interface. Let’s dive into it.

Duck Typing

To understand Go’s interfaces, it’s crucial to grasp the Duck Typing concept.

So, what’s Duck Typing?
```

## 使用 `bufio.Reader`

Go 中的按行读取，一种方法是，我们可以通过 `bufio` 提供的 `Reader` 实现。

演示一个例子，通过 `Reader` 读取文件。

```go
reader := bufio.NewReader(file)
for {
    line, err := reader.ReadString('\n')
    if err == io.EOF {
        break
    }
    if err != nil {
        panic(err)
    }
    fmt.Print(line)
}
```

## 使用 `bufio.Scanner`

按行读取，Go 中最简单的实现方法是使用 `bufio` 包中的 `Scanner`。这个方法适用于大多数的常规文件，特别是文件由许多短行组成时。

### 示例代码

通过一个示例代码，我们即可了解 `bufio.Scanner` 的使用方法。

```go
file, err := os.Open("example.txt")
if err != nil {
    panic(err)
}
defer file.Close()

// 创建文件的扫描器，用于逐行读取文件
scanner := bufio.NewScanner(file)
// 循环，直到文件结束
for scanner.Scan() {
    // 处理每行的内容：打印
    fmt.Println(scanner.Text())
}

// 检查扫描过程中是否有错误发生
if err := scanner.Err(); err != nil {
    panic(err)
}
```

输出：

```
This post covers the Golang Interface. Let’s dive into it.

Duck Typing

To understand Go’s interfaces, it’s crucial to grasp the Duck Typing concept.

So, what’s Duck Typing?
```

这个例子中，我们基于打开的文件描述符，创建了一个 `bufio.Scanner` 变量 `scanner`，它通过逐行扫描祝我们读取文件，直到结束。


### 如何读取大行？

但是，如果遇到特别长的行怎么办呢？

因为，默认情况下，`bufio.Scanner` 缓冲区的最大大小是 64k，即无法处理超过 64KB 的行。

为了解决这个问题，我们要调整默认的缓冲区大小。

```go
const maxCapacity = 1024 * 1024 // 例如，1MB
buf := make([]byte, maxCapacity)
scanner.Buffer(buf, maxCapacity)
```

在 `scanner` 开始扫描前，加上这一段代码，我们就可以处理那些异常长的行了。

我在处理字幕文件时，遇到过这种异常长的行。如果你用过 whisper 的话，可能了解我说的是什么。

那除了直接读取整行，还有没有其他什么更好的方法呢？

在解决这个问题前，我们先聊聊 `bufio.Scanner` 的一些源码逻辑吧。

### 缓存逻辑

它是如何 `bufio.Scanner` 是如何读取文件的？它的缓存逻辑是什么呢？

默认，`bufio.Scanner` 内部有一个 `s.buf` 缓冲区，当我们调用 `scannder.Scan` 方法时，它会尝试从 `Reader` 中读取一个缓存大小的内容，在需要时也会自动扩容。

说明：我们前面的案例中，`file` 变量就是这个 reader。

它的实现代码在 `bufio.Scanner` 的 Scan 方法中实现。如果当缓冲区大小不足以容纳一个完整的 token，Scanner 会自动增加缓冲区的大小。

什么是完整 token？

我们在读取一行数据时，这一行数据就是完整 token。如果想修改这个完整 token 的自定义，也是可以修改的。

接下来，让我们实现 `bufio.Scanner` 按单词读取。

### 切割方式定义

`bufio.Scanner` 在 Go 中是一个非常灵活的工具，除了按行读取文件，还可用于按不同方式分割输入数据。我们要做的就是它对完整 token 的定义，也就是修改它的字节切割方式 - `Scanner.split` 函数。

默认情况下，Scanner 按行分割（ScanLines）。我们可以通过设置自定义的 Split 函数 来改变它的行为，使其按单词或其他任何方式分割数据。

下面是将 Scanner 设置为按单词读取数据的示例：

```go
const input = "This is a test. This is only a test."
scanner := bufio.NewScanner(strings.NewReader(input))

// 设置分割函数为按单词分割
scanner.Split(bufio.ScanWords)

// 逐个读取单词
for scanner.Scan() {
    fmt.Println(scanner.Text())
}

if err := scanner.Err(); err != nil {
    fmt.Fprintln(os.Stderr, "reading input:", err)
}
```

输出：

```bash
This
is
a
test.
This
is
only
a
test.
```

现在，无论多大的文件，我们都可以通过巧妙定义切割方式来避免一次性读取的缺点了。

如我前面介绍的超大字幕文件，通过句号分割即可。而我要做的定义一个按句号分割的函数。

按句号分割的函数，如下所示：

```go
func ScanSentences(data []byte, atEOF bool) (advance int, token []byte, err error) {
    // 如果我们处于 EOF 并且有数据，则返回剩余的数据
    if atEOF && len(data) > 0 {
        return len(data), data, nil
    }

    // 查找句号
    if i := bytes.IndexByte(data, '.'); i >= 0 {
        // 返回找到的句子（包括句号），以及下一个 token 的起始位置
        return i + 1, data[:i+1], nil
    }

    return 0, nil, nil
}
```

如果你查看 `ScanLines` 的源码，`ScanSentences` 基本就是它的复制版本，其实就改了个分割符，哈哈。


上面的代码中，我们通过 `ReaderString('\n')` 中指定 `\n` 分割符实现按行读取。

如果想自定义 `Reader` 的缓存大小，可通过 `bufio.NewReaderSize(size)` 指定。

相对于 `Scanner` 类型，`Reader` 更加底层，它提供更普遍的一些方法。

上面的例子中，`Reader` 没有像 `Scanner` 一样隐藏 `error`，而是直接返回，如我们在循环中通过识别文件结尾 `EOF` 结束文件读取。

而且，由于 `Reader` 没有内置字符串分割的设计，如果要实现复杂的分割逻辑，如按单词分割，甚至是更复杂的分割逻辑，就要单独设计。

不过相对于 `Scanner`，它也有个易用点：按行读取无大小限制，即大行读取不会受缓存大小限制。它的底层逻辑是，如果一次读取没发现分隔符，会多次读取直到找到分隔符，然后将多次读取内容拼接返回。

## 结论

在 Go 中按行读取文件是一个常见且重要的任务。


