---
title: "Go 实现词法分析与解析 Part Three"
date: 2019-07-29T17:10:23+08:00
draft: false
comment: true
tags: ["Golang"]
---


# 译者前言

最近发现我的翻译是越来越随性了，刚开始文章翻译的时候比较拘束，现在更多强调可读性，比如有些对文章大意没有什么影响的文字我现在都会选择直接跳过。

这篇文章主要是关于 INI 解释器的 parser 实现，它会从上一节中 Lexer 中接收 Token 解析，最终返回给使用者具有实际意义的结构体。读了这个系列的文章，我相信大家对词法器实现的原理将会有了基本的理解，但如果要真正实践，似乎还有一段距离。有兴趣的话，我们可以实现个自己的 JSON 解释器。要求可以稍微简化，只解析到 JSON 的第一层。

译文如下：

---

本系列[第一篇文章](https://zhuanlan.zhihu.com/p/74474227)，[英文原版](https://adampresley.github.io/2015/06/01/writing-a-lexer-and-parser-in-go-part-3.html)，我们介绍了词法分析解析的一些基础概念，了解了 INI 文件的基本组成，并在此基础上定义了一些常量和结构体，这对我们接下来实现 INI 文件解析会很有帮助。

[第二篇文章](https://zhuanlan.zhihu.com/p/75324725)，[英文原版](https://adampresley.github.io/2015/05/12/writing-a-lexer-and-parser-in-go-part-2.html)，因主要聚焦在 Lexer 的实现。它完成了将输入文本转化为 Token 的过程。

今天是本系列的最后一篇文章，最终完成我们的解释器。解释器负责从 channel 读取 Token，并最终创建表示 INI 文件内容的结构体实例。解析完成后，我们可以用 JSON 格式将结果打印出来。

# 结构体

解析器负责启动词法器和从 channel 读取 Token 的组件。接收到 Token 后，解析器需要知道当前 Token 状态，然后将其解析到对应结构中。我们要做的第一件事就是，定义表示 INI 内容的结构体。将主要涉及三个结构体。

第一个表示 Key/Value 的结构体，名称为 IniKeyValue，如下。

**model/ini/IniKeyValue.go**
```go
package ini

type IniKeyValue struct {
	Key   string `json:"key"`
	Value string `json:"value"`
}
```

第二个表示 Section 的结构体，名称为 IniSection，如下：

**/model/ini/IniSection.go**

```go
package ini

type IniSection struct {
	Name          string        `json:"name"`
	KeyValuePairs []IniKeyValue `json:"keyValuePairs"`
}
```

我们知道，Section 是由 Key/Value 组成的，其中 KeyValuePairs 即是属于这个 Section 的 Key/Value。如果 Key/Value 是不属于任何 Section，将会属于 Name 为空的 Section 中。

最后一个表示整个文件的结构体，名称为 IniFile，如下：

**/model/ini/IniFile.go**
```go
package ini

type IniFile struct {
	FileName string       `json:"fileName"`
	Sections []IniSection `json:"sections"`
}
```

IniFile 有两个成员字段组成，分别 FileName 文件名称和一系列 Section。

# 解析器

解析器的编写，我们要做的第一件事是，创建一个用于存放解析结构的变量，即一个 IniFile 结构体类型变量。如下：

```go
output := ini.IniFile{
   FileName: fileName,
   Sections: make([]ini.IniSection, 0),
}
```

现在，我们还需要一些变量追踪 Token、 Token 的值，还有解析器当前的状态。例如，当我们在获取 Key/Value 时，我们需要知道当前处于哪个 Section 中。接下来，就可以启动词法器了。

```go
var token lexertoken.Token
var tokenValue string

/* State variables */
key := ""

log.Println("Starting lexer and parser for file", fileName, "...")

l := lexer.BeginLexing(filename, input)
```

到此，相关变量已经定义完成，并且词法器也成功启动。接下来开始从 channel 接收 Token，如果 Token 类型不是 TOKEN_VALUE，执行 Trim 空格。

```go
for {
	token = l.NextToken()

	if token.Type != lexertoken.TOKEN_VALUE {
		tokenValue = strings.TrimSpace(token.Value)
	} else {
		tokenValue = token.Value
	}

	...
```

我们知道，如果词法器遍历到文件结尾，将返回类型为 EOF 的 Token。此时，需要将 Section 和 Key/Value 记录下来，并退出循环。

```go
if isEOF(token) {
	output.Sections = append(output.Sections, section)
	break
}
```

parser 函数的最后实现具体 Token 的处理，主要有三个 Token 需要关注。

第一个是 Section Token，如果遇到 Section Token，我们首先检查 section 变量中是否已存在 Key/Value，有则将它记录到 output.Sections 中。然后重置 section 变量追踪当前 Section 和接下来 Key/Value。

接着是 Key/Value。如果遇到 TOKEN_KEY，变量 key 用于记录 TOKEN_KEY 的值。遇到 TOKEN_VALUE，我们就可以变量 key 对应的 value，并将其 append 到 section.KeyValuePairs 中。

示例代码如下：

```go
switch token.Type {
   case lexertoken.TOKEN_SECTION:
      /*
* Reset tracking variables
*/
      if len(section.KeyValuePairs) > 0 {
         output.Sections = append(output.Sections, section)
      }

      key = ""

      section.Name = tokenValue
      section.KeyValuePairs = make([]ini.IniKeyValue, 0)

   case lexertoken.TOKEN_KEY:
      key = tokenValue

   case lexertoken.TOKEN_VALUE:
      section.KeyValuePairs = append(section.KeyValuePairs, ini.IniKeyValue{
         Key: key,
         Value: tokenValue,
      })
      key = ""
   }
```

# 测试

开发工作已经完成，下面进入测试阶段。Github 上有相应的测试代码，go get 下载好代码，在你的 GOPATH 的 src/github.com/adampresley/sample-ini-parser 目录下的 sampleIniParser.go 即是测试代码。

代码如下：

```go
sampleInput := `
key=abcdefg

[User]
userName=adampresley
keyFile=~/path/to/keyfile

[Servers]
server1=localhost:8080
`

parsedINIFile := parser.Parse("sample.ini", sampleInput)
prettyJSON, err := json.MarshalIndent(parsedINIFile, "", " ")

if err != nil {
   log.Println("Error marshalling JSON:", err.Error())
   return
}

log.Println(string(prettyJSON))
```

直接通过 go run sampleIniParser.go 执行即可。执行输出如下：

```json
2019/08/01 00:06:33 Starting lexer and parser for file sample.ini ...
2019/08/01 00:06:33 Parser has been shutdown
2019/08/01 00:06:33 {
   "fileName": "sample.ini",
   "sections": [
      {
         "name": "",
         "keyValuePairs": [
            {
               "key": "key",
               "value": "abcdefg"
            }
         ]
      },
      {
         "name": "User",
         "keyValuePairs": [
            {
               "key": "userName",
               "value": "adampresley"
            },
            {
               "key": "keyFile",
               "value": "~/path/to/keyfile"
            }
         ]
      },
      {
         "name": "Servers",
         "keyValuePairs": [
            {
               "key": "server1",
               "value": "localhost:8080"
            }
         ]
      }
   ]
}
```

# 总结

这个系列文章是非常有挑战性，但也非常有趣。词法分析与解析是一个非常复杂的话题，有太多内容需要学习。我们可以看到，即使像上面 INI 文件解析这样简单的工作，我们也需要花费一些精力才能完成。

我的博文：[](https://www.poloxue.com/posts/2019-07-29-golang-lexer-and-parser-part3)，译：[Writing a Lexer and Parser in Go - Part 3](https://adampresley.github.io/2015/06/01/writing-a-lexer-and-parser-in-go-part-3.html)
