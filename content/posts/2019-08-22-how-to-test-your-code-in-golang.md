---
title: "如何测试你的 Go 代码"
date: 2019-08-22T13:01:20+08:00
draft: false
comment: true
tags: ["Golang"]
---

不论是开源项目，还是日常程序的开发，测试都是必不可少的一个环节。今天我们开始进入 Go 测试模块 testing 的介绍。

# 简单概述

我们选择开源项目，通常会比较关注这个项目的测试用例编写的是否完善，一个优秀项目的测试一般写的不会差。为了日后自己能写出一个好的项目，测试这块还是要好好学习下。

常接触的测试主要是单元测试和性能测试。毫无意外，go 的 testing 也支持这两种测试。单元测试用于模块测试，而性能则是由基准测试完成，即 benchmark。

Go 测试模块除了上面提到的功能，还有一项能力，支持编写案例，通过与 godoc 的结合，可以非常快捷地生成库文档。

# 最易想到的方法

谈到如何测试一个函数的功能，对开发来说，最容易想到的方法就是在 main 中直接调用函数判断结果。

举个例子，测试 math 方法下的绝对值函数 Abs，示例代码如下：

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	v := math.Abs(-10)
	if v != 10 {
		fmt.Println("测试失败")
		return
	}

	fmt.Println("测试成功")
}
```

更常见的可能是，if 判断都没有，直接 Print 输出结果，我们观察结果确认问题。特别对于习惯使用 Python、PHP 脚本语言的开发， 建一个脚本测试是非常快速的，因为曾经很长一段时间，我就是如此。

这种方式有什么缺点？我的理解，主要几点，如main 中的测试不容易复用，常常是建了就删；测试用例变多时，灵活性不够，常会有修改代码的需求；自动化测试也不是非常方便等等问题。

我对测试的了解不是很深，上面这些仅仅我的一些体验吧。

遇到了问题就得解决，下面正式开始进入 go testing 中单元测试的介绍。

# 一个快速体验案例

单元测试用于在指定场景下，测试功能模块在指定的输入情况下，确定有没有按期望结果输出结果。

我们直接看个例子，简单直观。测试 math 下的 Abs 绝对值函数。首先，在某个目录创建测试文件 math_test.go，代码如下：

```go
package math

import (
	"math"
	"testing"
)

func TestAbs(t *testing.T) {
	var a, expect float64 = -10, 10

	actual := math.Abs(a)
	if actual != expect {
		t.Fatalf("a = %f, actual = %f, expected = %f", a, actual, expect)
	}
}
```

程序非常简洁，a 是 Abs 函数的输入参数，expect 是期望得到的执行结果，actual 是函数执行的实际结果，测试结果由 actual 和 expect 比较结果确定。

完成用例编写，go test 命令执行测试，我们会看到如下输出。

```bash
$ go test
PASS
ok      study/test/math 0.004s
```

输出为 PASS，表示测试用例成功执行。0.004s 表示用例执行时间。

# 学会使用 go testing

从前面例子中可以了解到，Go 的测试写起来还是非常方便的。关于它的使用方式，主要有两点，一是测试代码的编写规则，二是 API 的使用。

## 测试的编写规则

Go 的测试必须按规则方式编写，不然 go test 将无法正确定位测试代码的位置，主要三点规则。

首先，测试代码文件的命名必须是以 _test.go 结尾，比如上节中的文件名 math_tesh.go 并非随意取的。

还有，代码中的用例函数必须满足匹配 TestXxx，比如 TestAbs。

关于 Xxx，简单解释一下，它主要传达两点含义，一是 Xxx 表示首个字符必须大写或数字，简单而言就是可确定单词分隔，二是首字母后的字符可以是任意 Go 关键词合法字符，如大小写字母、下划线、数字。

第三，关于用例函数类型定义，定义如下。

```go
func TestXxx(*testing.T)
```

测试函数必须按这个固定格式编写，否则 go test 将执行报错。函数中有一个输入参数 t, 类型是 *testing.T，它非常重要，单元测试需通过它反馈测试结果，具体后面再介绍。

## 灵活记忆 API 的使用

按规则编写测试用例只能保证 go test 的正确定位执行。但为了可以分析测试结果，我们还需要与测试框架进行交互，这就需要测试函数输入参数 t 的参与了。

在 TestAbs 中，我们用到了 t.Fatalf，它的作用就是反馈测试结果。假设没有这段代码，发生错误也会反馈测试成功，这显然不是我们想要的。

我们可以通过官方文档，看下 testing.T 中支持的可导出方法，如下：

```go
// 获取测试名称
method (*T) Name() string
// 打印日志
method (*T) Log(args ...interface{})
// 打印日志，支持 Printf 格式化打印
method (*T) Logf(format string, args ...interface{})
// 反馈测试失败，但不退出测试，继续执行
method (*T) Fail()
// 反馈测试成功，立刻退出测试
method (*T) FailNow()
// 反馈测试失败，打印错误
method (*T) Error(args ...interface{})
// 反馈测试失败，打印错误，支持 Printf 的格式化规则
method (*T) Errorf(format string, args ...interface{})
// 检测是否已经发生过错误
method (*T) Failed() bool
// 相当于 Error + FailNow，表示这是非常严重的错误，打印信息结束需立刻退出。
method (*T) Fatal(args ...interface{})
// 相当于 Errorf + FailNow，与 Fatal 类似，区别在于支持 Printf 格式化打印信息；
method (*T) Fatalf(format string, args ...interface{})
// 跳出测试，从调用 SkipNow 退出，如果之前有错误依然提示测试报错
method (*T) SkipNow()
// 相当于 Log 和 SkipNow 的组合
method (*T) Skip(args ...interface{})
// 与Skip，相当于 Logf 和 SkipNow 的组合，区别在于支持 Printf 格式化打印
method (*T) Skipf(format string, args ...interface{})
// 用于标记调用函数为 helper 函数，打印文件信息或日志，不会追溯该函数。
method (*T) Helper()
// 标记测试函数可并行执行，这个并行执行仅仅指的是与其他测试函数并行，相同测试不会并行。
method (*T) Parallel()
// 可用于执行子测试
method (*T) Run(name string, f func(t *T)) bool
```

上面列出了单元测试 testing.T 中所有的公开方法，我个人思路，把它们大概分为三类，分别是底层方法、测试反馈，还有一些其他运行控制的辅助方法。

基础信息的 API 只有 1 个，Name() 方法，用于获取测试名称。运行控制的辅助方法主要指的是 Helper、t.Parallel 和 Run，上面的注释对它们已经做了简单介绍。

我们这里重点说说测试反馈的 API，毕竟它用的最多。前面用到的 Fatalf 方法就是其中之一，它的效果是打印错误日志并立刻退出测试。希望速记这类 API 吗？我们或许可以按几个层级进行记忆。

首先，我们记住一些相关的基础方法，它们是其它方法的核心组成，如下：

- 日志打印，Log 与 Logf，Log 和 Logf 区别可对比 Println 和 Printf，即 Logf 支持 Printf 格式化打印，而 Log 不支持。
- 失败标记，Fail 和 FailNow，Fail 与 FailNow 都是用于标记测试失败的方法，它们的区别在于 Fail 标记失败后还会继续执行执行接下来的测试，而 FailNow 在标记失败后会立刻退出。
- 测试忽略，SkipNow 方法退出测试，但并不会标记测试失败，可与 FailNow 对比记忆。

我们再看看剩余的那些方法，基本都是由基础方法组合而来。我们可根据场景，选择不同的组合。比如：

- 普通日志，只是打印一些日志，可以直接使用 Log 或 Logf 即可；
- 普通错误，如果不退出测试，只是打印一些错误提示信息，使用 Error 或 Errorf，这两个方法是 log 或 logf 和 Fail 的组合；
- 严重错误，需要退出测试，并打印一些错误提示信息，使用 Fatal (log + FailNow) 或 Fatalf (logf + FailNow)；
- 忽略错误，并退出测试，可以使用 Skip (log + SkipNow) 和 Skipf (logf + SkipNow)；

如果支持 Printf 的格式化信息打印，方法后面都会有一个 f 字符。如此一总结，我们发现 testing.T 中的方法的记忆非常简单。

突然想到，不知是否有人会问什么情况下算是测试成功。其实，只要没有标记失败，测试就是成功的。

# 实践一个案例

讲了那么多基础知识，我都有点口感舌燥了。现在，开始尝试使用一下它吧！

举一个简单的例子，测试一个除法函数。首先，创建一个 math.go 文件。函数代码如下：

```go
package math

import "errors"

func Division(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}

	return a / b, nil
}
```

Division 非常简单，输入参数 a、b 分别是被除数和除数，输出参数是计算结果和错误提示。如果除数是 0，将会给出相应的错误提示。

在正式测试 Division 函数前，我们先要梳理下什么样的输入与期望结果表示测试成功。输入不同，期望结果也就不同，可能是正确结果，亦或者是期待的错误结果。什么意思？以这里的 Division 为例，两种场景需要考虑：

- 正常调用返回结果，比如当被除数为 10，除数为 5，期望得到的结果为 2，即期望得到正确的结果；
- 期望错误返回结果，当被除数为 10，除数为 0，期望返回除数不能为 0 的错误，即期望返回错误提示；

如果是测试驱动开发，在我们正式写实现代码前，就需要把这些先定义好，并且写好测试代码。

分析完用例就可以开始写代码啦。

先是正常调用的测试，如下：

```go
func TestDivision(t *testing.T) {
	var a, b, expect float64 = 10, 5, 2

	actual, err := Division(a, b)
	if err != nil {
		t.Errorf("a = %f, b = %f, expect = %f, err %v", a, b, expect, err)
		return
	}

	if actual != expect {
		t.Errorf("a = %f, b = %f, expect = %f, actual = %f", a, b, expect, actual)
	}
}
```

定义了三个变量，分别是 a、b、expect，对应被除数、除数和期望结果。用例通过对比 Division 的实际结果 actual 与期望结果 expect 确认测试是否成功。还有就是，Division 返回的 error 也要检查，因为这里期待的正常运行结果，只要有错即可认定测试失败。

再看期望错误结果，如下：

```go
func TestDivisionZero(t *testing.T) {
	var a, b float64 = 10, 0
	var expectedErrString = "division by zero"

	_, err := Division(a, b)
	if err.Error() != expectedErrString {
		t.Errorf("a = %f, b = %f, err %v, expect err %s", a, b, err, expectedErrString)
		return
	}
}
```

同样是首先定义了三个变量，a、b 和 expectErrString，a、b 含义与之前相同，expectErrString 为预期提示的错误信息。除数 b 设置为 0 ，主要是为了测试 Division 函数是否能按预期返回错误，所以我们并不关心计算结果。测试成功与否，通过比较实际的返回 error 与 expectErrString 确定。

通过 go test 执行测试，如下：

```bash
$ go test -v
=== RUN   TestDivision
--- PASS: TestDivision (0.00s)
=== RUN   TestDivisionZero
--- PASS: TestDivisionZero (0.00s)
PASS
ok      study/test/math 0.005s
```

结果显示，测试成功！

这个案例的演示中，我们在 go test 上加入 -v 选项，这样就可以清晰地看到每个测试用例的执行情况。

# 简洁紧凑的表组测试

通过上面的例子，不知道有没有发现一个问题？

如果将要测试的某个功能函数的用例非常多，我们将会需要写很多代码重复度非常高的测试函数，因为对于单元测试而言，基本都是围绕一个简单模式：

**指定输入参数** -> **调用要测试的函数** -> **获取返回结果** -> **比较实际返回与期望结果** -> **确认测试失败提示**

基于此，Go 提倡我们使用一种称为 "Table Driven" 的测试方式，中文翻译，可称为表组测试。它可以让我们以一种短小紧密的方式编写测试。具体如何做呢？

首先，我们要定义一个用于表组测试的结构体，其中要包含测试所需的输入与期望的输出。以 Division 函数测试为例，可以定义如下的结构体：

```go
type DivisionTable struct {
	a         float64 // 被除数
	b         float64 // 除数
	expect    float64 // 期待计算值
	expectErr error   // 期待错误字符串
}
```

各字段的含义在注释部分已经做了相关说明，和我们之前做的单个场景的测试涉及字段差不多。区别在于 expectErr 不再是 string 类型。

接下来，将下面我们需要测试的用例通过该结构体字面量表示出来。

```go
var table = []DivisionTable{
	{1., 1., 1., nil},
	{-4., -2., 2., nil},
	{2., 0., 7., errors.New("division by zero")},
}
```

简单列举了三种场景，分别正数之间的除法、负数之间的除数以及除数为 0 的情况下的除法。

接下来的目标就是实现一个通用 Division 测试函数。直接看代码吧！

```go
func TestDivisionTable(t *testing.T) {
	for _, v := range divisionTable {
		actual, err := Division(v.a, v.b)
		if err == nil {
			if v.expectErr != nil {
				t.Errorf(
					"a = %f, b = %f, actual err not nil, expect err is nil", v.a, v.b)
			}
		} else if err != nil {
			if v.expectErr == nil {
				t.Errorf(
					"a = %f, b = %f, actual err not nil, expect err is nil", v.a, v.b)
			} else if !strings.Contains(err.Error(), v.expectErr.Error()) {
				t.Errorf(
					"a = %f, b = %f, actual err = %v, expect err = %v", v.a, v.b, err, v.expectErr)
			}
		} else if actual != v.expect {
			t.Errorf(
				"a = %f, b = %f, actual = %f, expect = %f", v.a, v.b, actual, v.expect)
		}
	}
}
```

代码看起来比较乱，这主要是因为 error 接口内部实际类型是指针，不能直接使用比较操作符对比 error，所以要做一些处理。如果没有错误的比较，这个例子就容易理解的多了。

可以梳理一下，逻辑其实非常简单。主要由几个步骤组成：

- 首先遍历 divisionTable，获取到输入参数与期望结果；
- 使用从 divisionTable 获取到输入参数调用功能函数；
- 获取功能函数的执行结果，包括计算结果与可能的错误；
- 对比返回 err 与 expectErr，需要处理 err == nil 和 != nil 两种情况；
  - 实际 err 为 nil， expectErr 也需要为 nil；
  - 实际 err 不为 nil，expectErr 不可为 nil，且错误信息包含在 err 中。
- 最后一步，比较实际计算结果与期望结果；

如果发生错误，我们使用 t.Errorf 打印错误日志并告知测试失败。用 Errorf 的原因是我们不能只是一个用例失败就退出整个测试。当然，这个要视情况而定吧，没有固定规则。

介绍到这，核心部分就讲的差不多了。

# 详细的日志输出

对于一些刚接触 Go testing 的朋友，可能碰到过接下来要讲的这个奇怪的问题。

例子说明最直观，如下：

```go
func TestDivision(t *testing.T) {
	var a, b, expect float64 = 10, 5, 2

	actual, err := Division(a, b)
	if err != nil {
		t.Errorf("a = %f, b = %f, expect = %f, err %v", a, b, expect, err)
		return
	}

	if actual != expect {
		t.Errorf("a = %f, b = %f, expect = %f, actual = %f", a, b, expect, actual)
	}

	t.Log("end")
}
```

还是之前的例子，相比之下，最后增加了一段日志打印 t.Log("end")。不加任何选项的 go test 的执行效果如下：

```bash
$ go test
PASS
ok      study/test/math 0.004s
```

输出日志中并没看到增加那行 end 日志。

前面的演示中，我们用到了 go test 的 -v 选项，通过它，可以查看非常详细的输出信息。我们加上 -v 选项，再执行看效果：

```bash
$ go test -v
=== RUN   TestDivision
--- PASS: TestDivision (0.00s)
    math_test.go:36: end
PASS
ok      study/test/math 0.005s
```

多出了很多信息，并且打出了那行 end 日志，并给出代码的位置 math_test.go:36: end。除此以外，还具体到了每一个测试的执行情况，比如测试执行开始和测试结果。

# 灵活控制运行哪些测试

假设，我们把前面演示用到的那些测试函数全部放在 math_test.go 中。此时，使用默认 go test 测试会遇到一个问题，那就是每次都将包中的测试函数都执行一遍。有什么办法能灵活控制呢？

可以先来看看此类问题，常见的使用场景有哪些！我想到的几点，如下：

- 执行 package 下所有测试函数，go test 默认就是如此，不用多说；
- 执行其中的某一个测试函数，比如当我们把前面写的所有测试函数都放在了 math_test.go 文件中，如何选择其中一个执行；
- 按某一类匹配规则执行测试函数，比如执行名称满足以 Division 开头的测试函数；
- 执行项目下的所有测试函数，一个项目通常不止一个包，如何要将所有包的测试函数都执行一遍，该如何做呢；

第一个本不怎么用介绍了。但有一点还是要介绍下，那就是除默认执行当前路径的包，我们也可以具体指定执行哪个 package 的测试函数，指定方式支持纯粹的文件路径方式以及包路径方式。

假设，我们包的导入路径为 example/math，而我们当前位置在 example 目录下，就有两种方式执行 math 下的测试。

```
$ go test # 目录路径执行
$ go test example/math # GOPATH 包导入路径
```

第二、三场景，执行其中的某个或某类测试，主要与 go test 的 -run 选项有关，-run 选项接收参数是正则表达式。

执行某一个具体的函数，如 TestDivision，命令执行效果如下：

```bash
$ go test -run "^TestDivision$" -v
=== RUN   TestDivision
--- PASS: TestDivision (0.00s)
    math_test.go:36: end
PASS
ok      study/test/math 0.004s
```

从输出中可了解到，确实只执行了 TestDivision。这里要记住加上 -v 选项，使输出信息具体到某一个测试。

执行具体的某一个类的函数，如除法相关测试 Division，命令执行效果如下：

```bash
$ go test -run "Division" -v
=== RUN   TestDivision
--- PASS: TestDivision (0.00s)
    math_test.go:36: end
=== RUN   TestDivisionZero
--- PASS: TestDivisionZero (0.00s)
=== RUN   TestDivisionTable
--- PASS: TestDivisionTable (0.00s)
PASS
ok  	_/Users/polo/Public/Work/go/src/study/test/math	0.005s
```

将前面写过的函数名中包含 Division 全部执行一遍。

第四个场景，执行整个项目下的测试。在项目的顶层目录，直接执行 go test ./... 即可，具体就不演示了。

# 总结

本文主要介绍了 Go 中测试模块使用的一些基础方法。从最容易想到的通过 main 测试方法到使用 go testing 中编写单元测试，接着介绍了一个测试案例，由此引入了 Go 中推荐 "Table Driven" 的测试方式。最后，文章还介绍了一些对我们平时工作比较有实际意义的技巧。

博文地址：[如何测试你的 Go 代码](https://www.poloxue.com/how-to-test-your-code-in-golang)
# 参考

[Unit Testing make easy in Go](https://medium.com/rungo/unit-testing-made-easy-in-go-25077669318)  
[gotests](https://github.com/cweill/gotests)  
[go test 单元测试](http://objcoding.com/2018/09/14/go-test/)  
[testing - 单元测试](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter09/09.1.html)  
[Writing table driven tests in Go](https://dave.cheney.net/2013/06/09/writing-table-driven-tests-in-go)  
[How to write benchmarks in Go](https://dave.cheney.net/2013/06/30/how-to-write-benchmarks-in-go)  
[Go 如何写测试用例](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/11.3.md)  


