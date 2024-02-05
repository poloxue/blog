---
title: "Go 中如何解析 json 内部结构不确定的情况"
date: 2019-10-17T10:08:37+08:00
draft: false
comment: true
tags: ["Golang"]
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-10/2019-10-17-parse-dynamic-json-into-a-structure-01.png)

本文主要介绍的是关于 Go 如何解析 json 内部结构不确定的情况。

首先，我们直接看一个来提问吧。

问题如下：

上游传递不确定的json，如何透传给下游业务？比如，我解析参数

```json
{
    "test": 1,
    "key": {
        "k1": "1",
        "k2": 2
    }
}
```

但是key 结构体下面是未知的。可能是K1 K2 K3 ... KN。如何解析传递那？

对于 json 格式数据的解析，如果其中的某个成员结构不确定。

我总结一般有几种方式处理。

## 常见的几种方案

第一个方案，也是最容易想到的，将那个不确定的成员用 map[string]interface{} 替代。

```go
type Data struct {
    Test int                    `json:"test"`
    Key  map[string]interface{} `json:"test"`
}
```

但问题是，这种方式太坑，每次从 key 中拿数据，都要做类型检查，判断是否 ok。

第二种，既然 map[string]interface{} 的方式太坑，那如果要是能用结构体就好了。

虽然其中某个成员的结构不确定，但如果共性字段比较多，如都是与人相关，那肯定都有名字，年龄之类的字段，但如果是教师和学生，就会有一些不同的字段，把所有的不同字段都包含进来即可。但如果不同字段太多，那也不是很方便。

第三种，终极解决方案，如果能先解析第一层的结构，再根据第一层的结果，确定第二层的结构，那就方便多了。不确定的成员依然用 map[string]interface{} 表示，确定结构后，再将 map[string]interface{} 解析为具体的某个结构。结构体使用起来就方便很多了。

问题最终就变成了如何将 map[string]interface{} 转化为 struct，这个过程必然会用到反射，可以自己实现。但其他人早造就想到了，一个第三方库，地址：https://github.com/mitchellh/mapstructure 。

## 一个实际的案例

看一个我工作遇到的一个实际案例。

我在工作中，数据库数据实时更新到 elasticsearch，在实践过程中遇到了一些 JSON 数据处理的问题。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-10/2019-10-17-parse-dynamic-json-into-a-structure-02.png)

什么样的数据呢？

实时数据获取是通过 binlog 解析推送而来的的数据，并通过消息队列 kafka 传输给处理程序。

收到的 JSON，类似如下形式。

```json
{
    "type": "UPDATE",
    "database": "blog",
    "table": "blog",
    "data": [
        {
            "blogId": "100001",
            "title": "title",
            "content": "this is a blog",
            "uid": "1000012",
            "state": "1"
        }
    ]
}
```

简单说下数据的逻辑，type 表示数据库事件是新增、更新还是删除事件，database 表示对应的数据库名称，table 表示相应的表名称，data 即为数据库中数据。

怎么处理这串 JSON 呢？

## json 转化为 map

最先想到的方式就是通过 json.Unmarshal 将 JSON 转化 map[string]interface{}。

示例代码：

```go
func main () {
    msg := []byte(`{
    "type": "UPDATE",
    "database": "blog",
    "table": "blog",
    "data": [
        {
            "blogId": "100001",
            "title": "title",
            "content": "this is a blog",
            "uid": "1000012",
            "state": "1"
        }
    ]}`)
    var event map[string]interface{}
    if err := json.Unmarshal(msg, &event); err != nil {
        panic(err)
    }

    fmt.Println(event)
}
```

打印结果如下：

```
map[data:[map[title:title content:this is a blog uid:1000012 state:1 blogId:100001]] type:UPDATE database:blog table:blog]
```

到此，就成功解析出了数据。接下来是使用它，但我觉得 map 通常有几个不足。

- 通过 key 获取数据，可能出现不存在的 key，为了严谨，需要检查 key 是否存在；
- 相对于结构体的方式，map数据提取不便且不能利用 IDE 补全检查，key 容易写错；

针对这个情况，可以怎么处理呢？如果能把 JSON 转化为struct 就好了。

## json 转化为 struct

在 GO 中，json 转化为 struct 也非常方便，只需提前定义好转化的 struct 即可。我们先来定义一下转化的 struct。

```go
type Event struct {
	Type     string              `json:"type"`
	Database string              `json:"database"`
	Table    string              `json:"table"`
	Data     []map[string]string `json:"data"`
}
```

说明几点

- 实际场景中，canal 消息的 data 结构是由表决定的，在 JSON 成功解析前无法提前知道，所以这里定义为 map[string]string；
- 转化的结构体成员必须是可导出的，所以成员变量名都是大写，而与 JSON 的映射通过 `json:"tagName"` 的 tagName 完成。

解析代码非常简单，如下：

```go
e := Event{}
if err := json.Unmarshal(msg, &e); err != nil {
	panic(err)
}

fmt.Println(e)
```

打印结果：

```
{UPDATE blog blog [map[blogId:100001 title:title content:this is a blog uid:1000012 state:1]]}
```

接下来，数据解雇就方便了不少，比如事件类型获取，通过 event.Type 即可完成。不过，要泼盆冷水，因为 data 还是 []map[string]string 类型，依然有 map 的那些问题。

能不能把 map 转化为 struct ?

## map 转化为 struct

据我所知，map 转为转化为 struct，GO 是没有内置的。如果要实现，需要依赖于 GO 的反射机制。

不过，幸运的是，其实已经有人做了这件事，包名称为 [mapstructure](https://github.com/mitchellh/mapstructure)，使用也非常简单，敲一遍它提供的几个[例子](https://github.com/mitchellh/mapstructure/blob/master/mapstructure_examples_test.go)就学会了。README 中也说了，该库主要是遇到必须读取一部分 JSON 才能知道剩余数据结构的场景，和我的场景如此契合。

安装命令如下：

```
$ go get https://github.com/mitchellh/mapstructure
```

开始使用前，先定义 map 将转化的 struct 结构，即 blog 结构体，如下：

```go
type Blog struct {
	BlogId  string `mapstructure:"blogId"`
	Title   string `mapstructrue:"title"`
	Content string `mapstructure:"content"`
	Uid     string `mapstructure:"uid"`
	State   string `mapstructure:"state"`
}
```

因为，接下来要用的是 mapstructure 包，所以 struct tag 标识不再是 json，而是 mapstructure。

示例代码如下：

```go
e := Event{}
if err := json.Unmarshal(msg, &e); err != nil {
	panic(err)
}

if e.Table == "blog" {
	var blogs []Blog

	if err := mapstructure.Decode(e.Data, &blogs); err != nil {
		panic(err)
	}

	fmt.Println(blogs)
}
```

event 的解析和前面的一样，通过 e.Table 判断是是否来自 blog 表的数据，如果是，使用 Blog 结构体解析。接下来通过 mapstructure 的 Decode 完成解析。

打印结果如下：

```bash
[{100001 title this is a blog 1000012 1}]
```

到此，似乎已经完成了所有工作。非也！

## 弱类型解析

不知道大家有没有发现一个问题，那就是 Blog 结构体中的所有成员都是 string，这应该是 canal 做的事情，所有的值类型都是 string。但实际上 blog 表中的 uid 和 state 字段其实都是 int。

理想的结构体定义应该是下面这样。

```
type Blog struct {
	BlogId  string `mapstructure:"blogId"`
	Title   string `mapstructrue:"title"`
	Content string `mapstructure:"content"`
	Uid     int32  `mapstructure:"uid"`
	State   int32  `mapstructure:"state"`
}
```

但是当把新的 Blog 类型代入之前的代码，会如下的错误。

```bash
panic: 2 error(s) decoding:

* '[0].state' expected type 'int32', got unconvertible type 'string'
* '[0].uid' expected type 'int32', got unconvertible type 'string'
```

提示类型解析失败。其实，这种形式的 json 在其他一些软类型语言中也会出现。

那如何解决这个问题？提两种解决方案

- 使用时进行转化，比如类型为 int 的数据，使用时可以用 strconv.Atoi 转化。
- 使用 mapstructure 提供的软类型 map 转化 struct 的功能；

显然，第一种方式太 low，转化的时候还要多一步错误检查。那第二种方式如何呢？

来看示例代码，如下：

```go
var blogs []Blog
if err := mapstructure.WeakDecode(e.Data, &blogs); err != nil {
	panic(err)
}

fmt.Println(blogs)
```

其实只需要把 mapstructure 的 Decode 替换成 WeakDecode 就行了，字如其意，弱解析。如此easy。

到此，才算完成！接下来的数据处理就简单很多了。如果想学习 mapstructure 的使用，敲敲源码中例子应该差不多了。

## 总结

本文由一个问题引出主题，如何处理不确定结构的 json 数据，开头提出了三种可行的解决方案，三种方案是逐层递进的。最终的方式需要依赖反射实现，当然同样的问题别人早就想到了，并开发了一个第三方包，mapstructure。

最后，本文通过一个实际的案例演示了 mapstructure 的使用。

感谢阅读，希望本文对你有所帮助。

我的博文：[Go 中如何解析 json 内部结构不确定的情况](https://www.poloxue.com/posts/2a019-10-17-parse-dynamic-json-into-a-structure)
