
---
title: "Go 实现词法分析与解析 Part One"
date: 2019-07-17T17:03:59+08:00
draft: false
comment: true
tags: ["Golang"]
---

一直对词法分析与解析的话题比较感兴趣，最近发现了好几篇相关的优秀文章，准备好好翻译和研究下。我的理解，词法分析与解析的应用还是比较广泛的，无论简单的配置文件、各种模板语言、还是我们每天在写编程语言都离不开它。

本篇文章一个系列文章的第一篇，主要介绍的是词法分析与解析的一些基础概念，包括什么是词法分析，什么是解析，Token 如何表示等等。

正文如下：

从今天开始，我将会用三篇文章介绍在 Go 中如何构建一个简单的词法分析与解释器。文中介绍的内容主要是基于 Rob Pike 在 2011 年关于 [Lexical Scanning In Go](http://cuddle.googlecode.com/hg/talk/lex.html#landing-slide) 的演讲。这个系列文章最终会包含一个功能完善的代码，它可用于 INI 类型文件的解析。

三篇文章涉及内容分别是：

> - [Go 实现词法分析与解析](https://www.poloxue.com/posts/2019-07-17-golang-lexer-and-parser-part1)，[译：Writing a Lexer and Parser in Go - Part 1](https://adampresley.github.io/2015/04/12/writing-a-lexer-and-parser-in-go-part-1.html)，如什么是词法分析、解析，以及案例的一部分介绍；
> - [Go 实现词法分析与解析](https://www.poloxue.com/posts/2019-07-17-golang-lexer-and-parser-part2))，[译：Writing a Lexer and Parser in Go - Part 2](https://adampresley.github.io/2015/05/12/writing-a-lexer-and-parser-in-go-part-2.html)；
> - [Go 实现词法分析与解析](https://www.poloxue.com/posts/2019-07-17-golang-lexer-and-parser-part3)，[译：Writing a Lexer and Parser in Go - Part 3](https://adampresley.github.io/2015/06/01/writing-a-lexer-and-parser-in-go-part-3.html)；

## 概要

词法分析与解析是个比较复杂的话题，但这并不意味着我们无法一点点剖析和掌握它。为了帮助大家更好地了解它，接下来，我将会构建一个简单的 INI 文件解析器。这个解析器输入的是文本字符串，返回的是经过结构化处理的结果，结果包含多个 Section 和 Key/Value。我将用 Go 实现它。

为什么选择 INI 文件？主要是因为它的简单性，结构容易理解。例如，下面就是一个简单的 INI 内容样例：

```ini
[SetionName]
key1=value 1
key2=value 2
```

样例中主要涉及了三个元素，充分理解它们对于我们如何设计 INI 解释器是非常有帮助的。

- 段: Sections
- 键: Keys
- 值: Values

Key/Value 属于 Section，每个 Section 可能不止一个 Key/Value，每个 INI 文件可以包含多个 Section。这是一种非常简单但非常高效的结构，特别适用于保存配置信息。

上面的内容将会被解析为结构化数据，我们可以提前看下处理后的数据的 JSON 格式，如下：

```json
{
  "FileName": "sample.ini",
  "Sections": [
    {
      "Name": "SectionName",
      "KeyValuePairs": [
        {
          "Key": "key1",
          "Value": "value 1"
        },
        {
          "Key": "key2",
          "Value": "value 2"
        }
      ]
    }
  ]
}
```

## 什么是词法分析

词法分析在 WIKI 中的定义是 "将字符串转化为一系列 Token 的过程，即，一系列有意义的字符串"。词法分析通常是在编译或运行之前执行。例如，PHP 是一种解释型语言，当你访问一个由 PHP 开发的站点，PHP 解释器将负责 PHP 代码的执行，并把生成的 HTML 返回给浏览器。PHP 代码先会经过词法分析得到一系列有意义的 Token。之后，PHP 解释器会按照这些 Token 执行接下来的操作，比如将 Token 结果缓存，以及执行具体工作等。

## 什么是 Token

Token 是用于描述与归类从文本中分解出来的元素的一种结构。例如，之前的例子中，section 可归类为 TOKEN_SECTION，key 可以归类为 TOKEN_KEY。这种结构通常被用于追踪元素类类别和值。比如，前面的例子中，名为 `SectionName` 的 section，在 Token 中的结构是如下表示：

```json
{
    "Type": TOKEN_SECTION,
    "Value": "SectionName"
}
```

解析器、解释器或编译器将会根据得到 Token 决定如何执行、编译或生成代码/数据。

## 什么是解析

词法分析器将输入文本拆分，并返回一系列结构化的 token。但 token 本身并没有什么价值，如此便引出了解析的概念。解析是指对 Tokens 进行语法分析的过程，它可以确保输入的文本的可用性和有意义的。

例如，下面的样例就是非可用 INI 段 section。

```INI
[SectionName]=Hi there
```

这段文本在经过词法分析后，将会得到一系列的 Token，它们将被用于 section、等于号和字符串的表示。这是词法分析的职责所在。而解析器则是决定它们是否有意义，即是否符合语法。对于 INI 格式而言，这些 Token 并不可用。我们实现的解析器将会从 channel 中接收 Token，创建相应的数据结构，包含 section 和 key/value。

## 逐步拆解

本文最后一个任务，定义下面在词法分析器中将会使用 Token 类型结构，Token 的名称和相关的类型。首先是 Token 的结构，这个结构将会贯穿我们的整个代码，它将会通过 channel 传递给解析器。

我们先来看下项目目录结构，可以查看 [github 仓库](https://github.com/adampresley/sample-ini-parser), 在我的 Mac 上，目录结构是 `~/code/go/src/github.com/adampresley/sample-ini-parser`，lexer 词法分析组件在 service/lexer 目录下。Token 结构的定义位于 `~/code/go/src/github.com/adampresley/sample-ini-parser/services/lexer/lexertoken` 目录下。

```go
package lexertoken

import (
  "fmt"
)

type Token struct {
  Type  TokenType
  Value string
}
```

该结构清晰的表示一个 Token 由类型和值组成的结构。你可以已经注意到这里引用了一个还未定义的类型 TokenType。现在，我们来定义一下：

```go
package lexertoken

type TokenType int

const (
  TOKEN_ERROR TokenType = itoa
  TOKEN_EOF

  TOKEN_LEFT_BRACKET
  TOKEN_RIGHT_BRACKET
  TOKEN_EQUAL_SIGN
  TOKEN_NEWLINE

  TOKEN_SECTION
  TOKEN_KEY
  TOKEN_VALUE
)
```

我们新定义了一个名为 TokenType 的类型（源自整型），并创建了所有可能的 Token 类型常量，它们都是从 INI 文件基础上拆解而来。

- 我们需要一种方式实现错误追踪，定义 TOKEN_ERROR 表示错误类型；

- 当到达文本结尾，我们用 TOKEN_EOF 表示；

- 段由左括号、文本、右括号三部分组成；
  - TOKEN_LEFT_BRACKET
  - TOKEN_SECTION
  - TOKEN_RIGHT_BRACKET

- Key/Value，以等于号进行分隔；
  - TOKEN_KEY
  - TOKEN_EQUAL_SIGN
  - TOKEN_VALUE

- Section 和 Key/Value 必须以换行符结尾，常量 TOKEN_NEWLINE；

最后，我们还需要了解上面部分的 Token 类型的文本表示。比如，词法器在分析 Key/Value 是，会在它们之间寻找等于号，此时，我们需要知道它的文本表示，以确认当前位置是否存在等于号。用常量表示这类 Token 文本是个不错主意。

```go
package lexertoken

const EOF rune = 0

const LEFT_BRACKET string = "["
const RIGHT_BRACKET string = "]"
const EQUAL_SIGN string = "="
const NEWLINE string = "\n"
```

## 接下来

在 [Part 2](https://adampresley.github.io/2015/05/12/writing-a-lexer-and-parser-in-go-part-2.html)，我们将会对词法分析部分进行更加深入的介绍，我们在前面定义的 Token 结构也将会被用到。

我的博文：[Go 实现词法分析与解析 Part One](https://www.poloxue.com/posts/2019-07-17-golang-lexer-and-parser-part1)，译：[Writing a Lexer and Parser in Go - Part 1](https://adampresley.github.io/2015/04/12/writing-a-lexer-and-parser-in-go-part-1.html)

