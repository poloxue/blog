---
title: "GO 中高效 int 转换 string 的方法与源码剖析"
date: 2024-01-20T16:11:18+08:00
draft: false
comment: true
description: "本文将从逐步介绍几种在 Go 中将 int 转换为 string 的常见方法，并重点分析这几种方法在性能上的特点。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-04.png)

> 嗨，大家好！本文是系列文章 Go 小技巧第一篇，系列文章查看：[Go 语言小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

Go 语言 中，将整数（int）转换为字符串（string）是一项常见的操作。

本文将从逐步介绍几种在 Go 中将 int 转换为 string 的常见方法，并重点剖析这几种方法在性能上的特点。另外，还会重点介绍 `FormatInt` 高效的算法实现。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-01.png)

## 使用 `strconv.Itoa`

最直接且常用的方法是使用 `strconv` 包中的 `Itoa` 函数。`Itoa` 是 “Integer to ASCII” 的简写，它提供了一种快速且简洁的方式实现整数到字符串之间的转换。

示例代码如下：

```go
package main

import (
    "strconv"
    "fmt"
)

func main() {
    i := 123
    s := strconv.Itoa(i)
    fmt.Println(s)
}
```

`strconv.Itoa` 是通过直接将整数转换为其 ASCII 字符串表示形式。这个过程中尽量减少了额外的内存分配，没有复杂逻辑。

##  使用 `fmt.Sprintf`

另一种方法是，使用 `fmt` 包的 `Sprintf` 函数。这个方法在功能上更为强大和灵活，因为它能处理各种类型并按照指定的格式输出。

示例代码如下：

```go
package main

import (
    "fmt"
)

func main() {
    i := 123
    s := fmt.Sprintf("%d", i)
    fmt.Println(s)
}
```

虽然 `fmt.Sprintf` 在功能上非常强大，但它的性能通常不如 `strconv.Itoa`。

为什么呢？

因为 `fmt.Sprintf` 内部使用了反射（reflection）确定输入值类型，并且在处理过程中涉及到更多的字符串拼接和内存分配。

## 使用 `strconv.FormatInt`

当需要更多控制或处理非 int 类型的整数（如 int64）时，可以使用 `strconv` 包的 `FormatInt` 函数。

```go
package main

import (
    "strconv"
    "fmt"
)

func main() {
    var i int64 = 123
    s := strconv.FormatInt(i, 10)  // 10 表示十进制
    fmt.Println(s)
}
```

`strconv.FormatInt` 提供了对整数转换过程的更细粒度控制，包括 base 的选择（例如，十进制、十六进制等）。

与 `strconv.Itoa` 类似，`FormatInt` 在性能上也非常可观，而且 `FormatInt` 提供了既灵活又高效的解决方案。

如果我们查看 `strconv.Itoa` 源码，会发现 `strconv.Itoa` 其实是 `strconv.FormatInt` 的一个特殊情况。

```go
// Itoa is shorthand for FormatInt(int64(i), 10).
func Itoa(i int) string {
    return FormatInt(int64(i), 10)
}
```

现在 int 转 string 的高性能源码剖析，就变成了重点剖析 `FormatInt`。

## FormatInt 深入剖析

基于 Go 1.21 版本的 `itoa.go` 源码，我们可以深入理解 `strconv` 包中整数到字符串转换函数的高效实现。

```go
func FormatInt(i int64, base int) string {
	if fastSmalls && 0 <= i && i < nSmalls && base == 10 {
		return small(int(i)) // 100 以内的十进制小整数，使用 small 函数转化
	}
  	_, s := formatBits(nil, uint64(i), base, i < 0, false) // 其他情况使用 formatBits
	return s
}
```

以下是对其核心部分的详细解读，将会突出了其性能优化的关键方面，结合具体的源码实现说明。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-02.png)


### 1. 快速路径处理小整数

对于常见的小整数，`strconv` 包提供了一个快速路径，small 函数，直接返回预先计算好的字符串，避免了运行时的计算开销。

```go
func small(i int) string {
	if i < 10 {
		return digits[i : i+1]
	}
	return smallsString[i*2 : i*2+2]
}
```

对于小于 100 的十进制整数，采用这个快速实现方案，或许这也是整数转字符串的最常见使用场景吧。

`small` 函数通过索引到 `smallsString`  和 `digits` 获取小整数的字符串表示，这个过程非常快速。
`digits` 和 `smallsString` 的值，如下所示：

```go
const smallsString = "00010203040506070809" +
	"10111213141516171819" +
	"20212223242526272829" +
	"30313233343536373839" +
	"40414243444546474849" +
	"50515253545556575859" +
	"60616263646566676869" +
	"70717273747576777879" +
	"80818283848586878889" +
	"90919293949596979899"

const digits = "0123456789abcdefghijklmnopqrstuvwxyz"
```

它们也就是十进制 0-99 与对应字符串的映射。

### 2. formatBits 函数的高效实现

FormatInt 最复杂的部分是 `formatBits` 函数，它是整数到字符串转换的核心，它针对不同的基数进行了优化。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-03.png)

**10进制转换的优化**

对于10进制转换，`formatBits` 使用了基于除法和取余的算法，并通过 `smallsString` 加速两位数的字符串获取。

```go
if base == 10 {
	// ... (32位系统的优化)
	us := uint(u)
	for us >= 100 {
		is := us % 100 * 2
		us /= 100
		i -= 2
		a[i+1] = smallsString[is+1]
		a[i+0] = smallsString[is+0]
	}
	// ... (处理剩余的数字)
}
```

- 对于 32 位系统，使用32位操作处理较大的数字，减少 64 位除法的开销。
- 每次处理两位数字，直接从 `smallsString` 获取对应的字符，避免了单独转换每一位的开销。

**2的幂基数的优化**

对于基数是2的幂的情况，`formatBits` 使用了位操作来优化转换。

```go
} else if isPowerOfTwo(base) {
	shift := uint(bits.TrailingZeros(uint(base))) & 7
	b := uint64(base)
	m := uint(base) - 1 // == 1<<shift - 1
	for u >= b {
		i--
		a[i] = digits[uint(u)&m]
		u >>= shift
	}
	// u < base
	i--
	a[i] = digits[uint(u)]
}
```

- 位操作是直接在二进制上进行，比除法和取余操作更快。
- 利用 2 的幂基数的特性，通过移位和掩码操作获取数字的各个位。

**通用情况的处理**

对于其他基数，`formatBits` 使用了通用的算法，但仍然尽量减少了除法和取余操作的使用。

```go
} else {
	// general case
	b := uint64(base)
	for u >= b {
		i--
		// Avoid using r = a%b in addition to q = a/b
		// since 64bit division and modulo operations
		// are calculated by runtime functions on 32bit machines.
		q := u / b
		a[i] = digits[uint(u-q*b)]
		u = q
}
```

我觉得最核心的算法就是利用移位和特殊路径预置映射关系。另外，由于算法足够优秀，还避免了一些不必要内存分配。

## 结论

将 int 转化为 string 是一个非常常见的需求。Go 语言的 `strconv` 包中的 int 到 string 的转换函数展示了 Go 标准库对性能的深刻理解和关注。

通过快速处理小整数、优化的 10 进制转换算法、以及2^n 基数的特别处理，这些函数能够提供高效且稳定的性能。这些优化确保了即使在大量数据或在性能敏感的场景中，`strconv` 包的函数也能提供出色的性能

博文地址：[GO 中高效 int 转换 string 的方法与源码剖析](https://www.poloxue.com/posts/2024-01-20-int-to-string-in-golang/)
