---
title: "Go 实现词法分析与解析 Part Two"
date: 2019-07-24T17:10:19+08:00
draft: false
comment: true
tags: ["Golang"]
---


本文是关于词法器实现的具体介绍，如果在阅读时遇到困难，建议参考源码阅读，文中的代码片段为了介绍思路。如何解析会在下一篇介绍。

最近简单看了下 Go 源码，在 src/go 目录下有几个模块，token、scanner 和 parser 应该就是 Go 词法相关实现的核心代码，打开 token 目录会发现其中的源码和上一节介绍的内容有诸多相似之处。

由于最近并发任务比较多，不能以最快的速度更新。词法的相关内容，除了本系列，我把其他一些相关文章的链接都贴在下面，如果英文阅读功底不错，可自行阅读。

[A look at Go lexer/scanner packages](https://arslan.io/2015/10/12/a-look-at-go-lexerscanner-packages/)  
[Rob Pike's Functional Way](https://www.youtube.com/watch?v=HxaD_trXwRE)  
[Handwritten Parser & Lexers In Go](https://blog.gopheracademy.com/advent-2014/parsers-lexers/)  

译文如下：

本系列的[第一篇文章](https://www.poloxue.com/posts/2019-07-17-golang-lexer-and-parser-part1/)（[英文原版](https://adampresley.github.io/2015/04/12/writing-a-lexer-and-parser-in-go-part-1.html)）。

我介绍了关于词法分析与解析的一些基本概念和 INI 文件内容的基本组成。之后，我们创建了部分相关结构体与常量，帮助实现接下来的 INI 文本解析器。

本篇文章将实际深入到词法分析的细节。

词法分析 (lexing)，指的是将输入文本转化为一系列 Token 的过程。Token 是比文本更小的单元，将它们组合在一起才可能产生有实际意义的内容，如程序、配置文件等。

本系列文章中的 INI 文件，Token 包括左括号、右括号、SectionName、Key，Value 以及等于号。用正确的顺序组合它们，你就会有一个 INI 文件。词法器的职责是读取 INI 文件内容、分析创建 Token，以及通过 channel 将 Token 发送给解析器。

# 词法分析器

为了实现文本到 Token 的转化，我们还需要追踪一些信息，比如文本内容，当前分析文本的位置，以及当前分析的 Token 的开始和结束位置。

完成分析后，我们还要将 Token 发送给解析器，可以通过 channel 传递。

我们还需要一个函数实现词法器状态的追踪。Rob Pike 的演讲中谈到利用函数追踪词法器当前和接下来期望的状态。简单而言，就是一个函数处理一个 Token，并返回下一个状态函数生成下一个期望 Token。下面，我就简单翻译为状态函数吧.

举个例子吧！

INI 中 Section 由三部分组成，分别是左括号、SectionName 以及右括号。第一个函数将会生成左括号类型的 Token，返回 SectionName 的状态函数，它会分析处理 SectionName 的相关逻辑，并返回处理右括号的状态函数。总的顺序是，左括号 -> section 名称 -> 右括号。

百闻不如意见，具体看下词法器的结构吧。如下：

**Lexer.go**

```go
type Lexer struct {
  Name   string
  Input  string  // 输入文本
  Tokens chan lexertoken.Token // 用于向词法分析器发送 Token 的 channel
  State  LexFn   // 上面提到的状态函数

  Start int      // token 的开始位置，结束位置可以通过 start + len(token) 获得
  Pos   int      // 词法器处理文本位置，当确认 Token 结尾时，即相当于知道 Token 的 end position
  Width int
}
```

**LexFn.go**

```go
type LexFn func(*Lexer) LexFn  // 词法器状态函数的定义，返回下一个期望 Token 的分析函数。
```

上篇文章，我们已经定义了 Token 结构。LexFn，是用于处理 Token 的词法器状态函数类型。

现在再为我们的额词法器增加一些能力。Lexer 是用于文本处理的，为了获取下一个 Token，我们为 Lexer 增加诸如读取 rune 字符串、跳过空格，和其他一些有用的方法。基本都是文本处理的一些简单方法。

```go
/*
Puts a token onto the token channel. The value of this token is
read from the input based on the current lexer position.
*/
func (this *Lexer) Emit(tokenType lexertoken.TokenType) {
    this.Tokens <- lexertoken.Token{Type: tokenType, Value: this.Input[this.Start:this.Pos]}
    this.start = this.Pos
}

/*
Increment the position
*/
func (this *Lexer) Inc() {
    this.Pos++
    if this.Pos >= utf8.RuneCountInString(this.Input) {
        this.Emit(lexertoken.TOKEN_EOF)
    }
}

/*
Return a slice of the input from the current lexer position
to the end of the input string.
*/
func (this *Lexer) InputToEnd() string {
    return this.Input[this.Post:]
}

/*
Skips whitespace until we get something meaningful
*/
func (this *Lexer) SkipWhiteSpace() {
    for {
        ch := this.Next()
        if !unicode.IsSpace(ch) {
            this.Dec()
            break
        }

        if ch == lexertoken.EOF {
            this.Emit(lexertoken.TOKEN_EOF)
            break
        }
    }
}
```

重点需要了解的是，Token 的读取与发送。主要涉及几个步骤，如下：

首先，一直读取字符，直到形成一个确定的 Token，举例说明，SectionName 的状态函数，只有读到右括号才能确认 SectionName。
接着，将 Token 和 Token 类型通过 channel 发送给解析器。
最后，判断下一个期望的状态函数，并返回。

我们先定义一个启动函数。它同样是解析器（下篇文章）的启动入口。它初始化了一个 Lexer，赋予它第一个状态函数。

第一个期望的 Token 可能是什么？一个特殊符号还是一个关键词？

在我们的例子中，第一个状态函数将会用一个通用的名称 LexBegin 命名，因为在 INI 文件中，section 开始可以，但也可以没有 section，以 key/value 开投。**LexBegin** 会负责处理这个逻辑。

```go
/*
Start a new lexer with a given input string. This returns the
instance of the lexer and a channel of tokens. Reading this stream
is the way to parse a given input and perform processing.
*/
func BeginLexing(name, input string) *lexer.Lexer {
    l := &lexer.Lexer{
        Name: name,
        Input: input,
        State: lexer.LexBegin,
        Tokens: make(chan lexertoken.Token, 3),
    }

    return l
}
```

# 开始

第一个状态函数 **LexBegin**。

```go
/*
This lexer function starts everything off. It determines if we are
beginning with a key/value assignment or a section.
*/
func LexBegin(lexer *Lexer) LexFn {
    lexer.SkipWhitespace()
    if strings.HasPrefix(lexer.InputToEnd(), lexertoken.LEFT_BRACKET) {
        return LexLeftBracket
    } else {
        return LexKey
    }
}
```

正如所见，首先是跳过所有空格，INI 文件中，空格是没有意义。接着，我们需要确认第一个字符是否是左括号，是的话，则返回 **LexLetBracket**，否则即是 key 类型，返回 **LexKey** 状态函数。

# Section

开始 section 的处理逻辑介绍。

INI 文件中的 SectionName 是由左右括号包裹起来的。我们可以将 Key/Value 组织在某个 Section 中。在 **LexBegin** 中，如果发现了左括号，则会返回 **LexLeftBracket** 函数。

**LexLeftBracket** 的代码如下：

```go
/*
This lexer function emits a TOKEN_LEFT_BRACKET then returns
the lexer for a section header.
*/
func LexLeftBracket(lexer *Lexer) LexFn {
    lexer.Pos += len(lexertoken.LEFT_BRACKET)
    lexer.Emit(lexertoken.TOKEN_LEFT_BRACKET)
    return LexSection
}
```

代码很简单！根据括号长度（长度位 1），将词法器的位置后移，接着向 channel 发送 TOKEN_LEFT_BRACKET。

在这个场景下，Token 内容并没有什么意义。当 Emit 执行完成后，开始位置被赋值为词法器当前位置，这将会为下一个 Token 做好准备。最后，返回用于处理 SectioName 的状态函数，**LexSection**。

```go
/*
This lexer function exits a TOKEN_SECTION with the name of an
INI file section header.
*/
func LexSection(lexer *Lexer) LexFn {
    for {
        if lexer.IsEOF() {
            return lexer.Errorf(errors.LEXER_ERROR_MISSING_RIGHT_BRACKET)
        }

        if strings.HasPrefix(lexer.InputEnd(), lexertoken.RIGHT_BRACKET) {
            lexer.Emit(lexertoken.TOKEN_SECTION)
            return LexRightBracket
        }

        lexer.Inc()
    }
}
```

逻辑稍微有点复杂，但基本逻辑一样。

函数中通过一个循环遍历字符，直到遇到 RIGHT_BRACKET，即右括号，才可以确认 SectionName 的结束位置。如果遇到 EOF，则说明是一个错误格式的 INI，我们应该进行错误提示，并通过 channel 发送给解析器。如果正常，将一直循环，直到发现右括号，然后 TOKEN_SECTION 和相应文本发送出去。

**LexSection** 返回的状态函数是 **LexerRightBracket**，逻辑与 **LexerLeftBracket** 类似，不同的是，它返回的状态函数是 **LexBegin**， 原因是 Section 可能是空 Section，也可能有 Key/Value。

```go
/*
This lexer function emits a TOKEN_RIGHT_BRACKET then returns
the lexer for a begin.
*/
func LexRightBracket(lexer *Lexer) LexFn {
    lexer.Pos += len(lexertoken.RIGHT_BRACKET)
    lexer.Emit(lexertoken.TOKEN_RIGHT_BRACKET)
    return LexBegin
}
```

# Key/Value

继续 Key/Value 处理的介绍，它的表达形式非常简单：key=value。

首先是 Key 的处理，和 **LexSection** 类似，一直循环直到遇到等于号才能确定一个完整的 Key。然后执行 Emit 将 Key 发送，并返回状态函数 **LexEqualSign**。

```go
/*
This lexer function emits a TOKEN_KEY with the name of an
key that will assigned a value
*/
func LexKey(lexer *Lexer) LexFn {
    for {
        if strings.HasPrefix(lexer.InputToEnd(), lexertoken.EQUAL_SIGN) {
            lexer.Emit(lexertoken.TOKEN_KEY)
            return LexEqualSign
        }

        lexer.Inc()
        if lexer.IsEOF() {
            return lexer.Errorf(errors.LEXER_ERROR_UNEXPECTED_EOF)
        }
    }
}
```

等号的处理非常简单，和左右括号类似。直接发送 **TOKEN_EQUAL_SIGN** 类型 Token 给解析器，并返回 **LexValue**。

```go
/*
This lexer functions emits a TOKEN_EQUAL_SIGN then returns
the lexer for value.
*/
func LexEqualSign(lexer *Lexer) LexFn {
    lexer.Pos += len(lexertoken.EQUAL_SIGN)
    lexer.Emit(lexertoken.EQUAL_SIGN)

    return LexValue
}
```

最后介绍的状态函数是 **LexValue**，用于 Key/Value 中的 Value 部分的处理。它会在遇到换行符时确认一个完整的Value。它返回的状态函数是 **LexBegin**，以此继续下一轮的分析。

```go
/*
This lexer function emits a TOKEN_VALUE with the value to be assigned
to a key.
*/
func LexValue(lexer *Lexer) LexFn {
    for {
        if strings.HasPrefix(lexer.InputToEnd(), lexertoken.NEWLINE) {
            lexer.Emit(lexertoken.TOKEN_VALUE)
            return LexBegin
        }

        lexer.Inc()

        if lexer.IsEOF() {
            return lexer.Errorf(errors.LEXER_ERROR_UNEXPECTED_EOF)
        }
    }
}
```

# 接下来

在 [Part 3](https://adampresley.github.io/2015/06/01/writing-a-lexer-and-parser-in-go-part-3.html)，本系列的最后一篇，我们将会介绍如何创建一个基本的解析器，将从 lexer 获得的 Token 处理为我们期望得到的结构化数据。

我的博文：[Go 实现词法分析与解析 Part Two](https://www.poloxue.com/posts/2019-07-24-golang-lexer-and-parser-part2)，译：[Writeing a Lexer and Parser in Go - Part 2](https://adampresley.github.io/2015/05/12/writing-a-lexer-and-parser-in-go-part-2.html)
