---
title: "Golang 中如何实现多行字符串"
date: "2023-10-10T22:00:36+08:00"
draft: false
comment: true
tags: ["Golang"]
---

Python 中，如果想要表示多行字符串，只要通过三单/双引号("""）包裹字符串即可。

类似代码，如下所示。

```python
a = """line1
line2
line3"""
print(a)
```

执行代码，查看输出效果，如下所示：

```bash
line1
line2
line3
```

Golang 中如何实现呢？不复杂，简单展示两种方式：

## 方式 1. 通过 ` 符号

具体代码如下：

```golang
str := `hello
world`
```

这种方式的性能最优。

## 方式 2. 通过 + 号进行拼接。

Golang 支持通过 + 拼接字符串，如下所示：

```
s := "hello" +
  "world"
fmt.Print(s)
```

输出如下所示：

```bash
helloworld
```

从输出结果已经看出来，没有换行效果。对于使用 + 拼接，需要使用 `\n` 转义符，进行换行。

代码如下所示：

```
s := "hello\n" + 
  "world"
fmt.Println(s)
```

输出如下所示：

```
hello
world
```

是我们期望的结果。

搞定！



