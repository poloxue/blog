---
title: "Go 中如何使用反射 Part Two"
date: 2019-08-17T17:25:03+08:00
draft: false
comment: true
tags: ["Golang"]
---


# 译者前言

这篇博文介绍的内容比较实在，主要是关于两方面的内容。一是介绍 reflection 在 encoding/json 中的应用，另一个开发了一个 Cacher 工厂函数，实现函数式编程中的记忆功能，其实就是根据输入对输出进行一定限期的缓存。

这篇文章的翻译没有上一篇那么轻松，因为涉及了一些函数式编程的术语，之前也并没有接触过。为了翻译这篇文章，简单阅读了网上的一篇关于函数式编程的文章，[文章地址](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)。望没有知识性错误。

译文如下：

----

上一篇[文章](https://juejin.im/post/5cf89f78f265da1b942139c3)，(阅读[英文原版](https://medium.com/capital-one-tech/learning-to-use-go-reflection-822a0aed74b7))，我们介绍了 Go 的反射包 reflection。并通过一些示例介绍了它的特性。但是，我们还不清楚它究竟有什么。

通过反射实现的功能，不用反射也能实现，而且更加高效简洁。但是 Go 团队肯定不会因为自己而为 Go 增加一个新的特性。

那究竟什么情况下会使用反射呢？

# 寻找反射使用案例

通过反射，我们可以实现各种奇淫巧技。但每天的工作中，我该如何使用它呢？

其实，大部分时间里，我们都用不到它。反射主要是用在一些特殊的场景下，使一些不可能的实现成为可能。我们常会在一些库、工具、框架中找到反射的使用场景。

那你是否可以告诉我，哪些库、框架或工具中使用反射呢？一个技巧，查看函数参数类型。如果一个函数的参数类型是 interface{}，那么，它极有可能使用了反射来检查或改变参数的值。

# JSON 处理

反射，最常见的使用场景之一，是对网络或文件中的数据进行解包和组包。当你通过 struct tag 映射 JSON 或数据库中的数据时，便是通过反射实现的。这类场景，我们通常会用某个库帮助我们创建结构体实例，它通过分析 struct tag 和数据，以此为 struct 的字段赋值。

我们就以 Go 官方标准库中的 JSON 解包为例，来介绍一下它的实现。

通过调用 json.Unmarshal 函数，我们可以把 JSON 字符串解包并赋值给某个变量。Unmarshal 函数接收两个参数：

- 类型为 []byte 的 JSON 字符串；
- 类型为 interface{}，用于存放 JSON 解析结果的变量；

深入看看这个函数究竟是如何进行反射的？

阅读 json 包的源码，其中有个私有函数 unmarshal，主要看其中与反射相关的部分代码如下：

```go
func (d *decodeState) unmarshal(v interface{}) (err error) {
    <skip some setup>
    rv := reflect.ValueOf(v)
    if rv.Kind() != reflect.Ptr || rv.IsNil() {
        return &InvalidUnmarshalError{reflect.TypeOf(v)}
    }
    d.scan.reset()
    // We decode rv not rv.Elem because the Unmarshaler interface
    // test must be applied at the top level of the value.
    // 传的是 rv，而不是 rv.Elem，因为结果传递给最顶层的 value
    d.value(rv)
    return d.savedError
}
```

上面的代码中，首先会通过反射验证变量类型，是否是指针类型，如果是，将变量 v 的 reflect.Value 传给 value 方法。

在 value 方法中，首先检查 JSON 字符串表示的类型，array、object、还是字面量。不同的类型将由不同方法处理。举例来说，如果解析 JSON Object。

将会有很多地方用到反射。

比如，使用反射检查 v 是否是 nil interface。

```go
// Decoding into nil interface? Switch to non-reflect code.
if v.Kind() == reflect.Interface && v.NumMethod() == 0 {
	v.Set(reflect.ValueOf(d.objectInterface()))
	return
}
```

如果是把 JSON object 赋值给 map。

```go
switch v.Kind() {
	case reflect.Map:
	// Map key must either have string kind, have an integer kind,
	// or be an encoding.TextUnmarshaler.
	t := v.Type()
	switch t.Key().Kind() {
		case reflect.String,
			reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64,
			reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
 		default:
 			if !reflect.PtrTo(t.Key()).Implements(textUnmarshalerType) {
 				d.saveError(&UnmarshalTypeError{Value: "object", Type: v.Type(), Offset: int64(d.off)})
 				d.off --
 				d.next() // skip over { } in input
 				return
 			}
 	}
 	if v.IsNil() {
 		v.Set(reflect.MakeMap(t))
 	}
 	case reflect.Struct:
 		// ok
 	default:
 		d.saveError(&UnmarshalTypeError{Value: "object", Type: v.Type(), Offset: int64(d.off)})
		d.off --
 		d.next() // skip over { } in input
 		return
}
```

如果是把 JSON object 赋值给 struct。

```go
subv = v
destring = f.quoted
for _, i := range f.index {
	if subv.Kind() == reflect.Ptr {
		if subv.IsNil() {
			subv.Set(reflect.New(subv.Type().Elem()))
		}
		subv = subv.Elem()
	}
	subv = subv.Field(i)
}
```

以上只是一个简单的演示，主要是关于，JSON 包中反射是如何使用的。如果希望自己阅读代码，源码在 [encoding/json/decode.go](https://github.com/golang/go/blob/master/src/encoding/json/decode.go)。

# 记忆与短期记忆

Json Unmarshal 是反射的其中一个使用案例。还有其他场景吗？接下来，我们将用反射开发一个库，基于 "记忆法" 实现短期缓存。

或许你并不熟悉这个术语，"记忆" 出自于函数式编程。函数式编程中倾向于强制推行某些规则，比如，参数和变量通常是不可修改的，创建之后便不可变。函数式编程尝试限制编程的 "副作用"。

一个实际的程序基本是不可能的没有 "副作用" 的，因为总会涉及诸如信息打印、写文件、向数据库插入数据等事情。但其他的一些 "副作用"，比如更新全局变量，将会导致程序很难追踪。函数式编程的一个目标，让程序中的数据追踪变得简单，我们可以非常容易地就明白程序在做什么？

函数式编程还有一些其他好处。比如，当一个函数输入和返回不变且没有 "副作用" 时，每次调用函数，相同的输入，做同样的工作，返回相同的结果。如果我们保存了这些结果，重复的工作就没有必要做第二次了。

到此，我们开始引入 "记忆" 的概念。它类似于函数级别的缓存。"记忆" 是通过在恒定的函数上包裹函数，实现输入输出的缓存，从而避免重复不必要的工作。但一个函数被 "记忆"，对于一个输入，只有执行一次工作。相同的输入执行第二次函数调用时，返回值将从缓存中获取，而不是重新计算一次。对于那些复杂或非常慢的操作，如此对性能的提升将是非常大的。

Go 可能不算是函数式编程语言，但我们依然可以通过它践行自己的想法。这种编程风格稍微有点严格，但是它避免了输入输出的更新，最小化了程序的 "副作用"，使你的程序非常易读。

我们将实现一小短时间的缓存，而不是永远。在微服务架构中，这是一种更加通用的模式。比如如下的场景：

一个服务提供数据，另一个服务获取数据。因为数据是通过网络传递的，会占用一定时间。这将会降低系统的整体性能。如果数据不是经常改变的且数据延迟几秒更新也没有关系，那么可以暂时缓存这个数据，它将给你的系统带来一个显著的性能提升。通过函数式编程的 "记忆"，我们可以在也没有对太多 API 的修改实现缓存，避免系统额外的网络调用。

如何实现？我们将通过反射实现三件事：

- 确保输入类型是一个函数，并且至少包含一个输入和一个输出；
- 确保新创建结构体中的字段类型和输入参数的类型一一对应；
- 确保新创建函数的输入和输出参数和原始函数相匹配；

还有一个限制，所有的输入参数必须可比较，在 Go 中，可比较的类型及可以通过 == 符号进行布尔运算。我们将用 Go 中的映射 map 关联输入和输出，Go 中 map 的 key 必须是可比较的，这很有意义，因为要确认输入参数是否出现过，我们需要检查前后的相等性。

幸运的是，Go 中只有四种情况是不可比较的，如下：

- 切片，Slices；
- 映射，Maps；
- 函数，Functions；
- 结构体，成员包含 Slice、Map 或 Function；

让我们开始定义 Cacher 函数，类似如下的形式：

```go
// Takes in a function and a time.Duration and returns a caching version of the function
// 接收两个参数，分别是函数和时间，返回的是一个缓存版本的函数
//
// The limitations on memoization are:
// 限制如下：
//
// — there must be at least one in parameter
// - 必须有一个输入参数
//
// — there must be at least one out parameter
// - 必须有一个输出参数
//
// — the in parameters must be comparable. That means they cannot be of kind slice, map, or func,
// nor can input parameters be structs that contain (at any level) slices, maps, or funcs.
// - 输入参数必须是可比较的，
//
// Be aware that if your memoized function has any side-effects (does anything that isn’t
// reflected in the output, like print to the screen or write to a database) the side-effects
// will be performed by the function only the first time that the function is invoked with
// particular set of values.
// 要明白了解，如果被记忆的函数有任何的副作用（指任何不能在返回结果中反应出来的东西，比如打印、写数据库），这些 "副作用"
// 将只会执行一次。
func Cacher(f interface{}, expiration time.Duration) (interface{}, error) {
	return f, nil
}
```

代码并没有做什么，但通过它，我们明白了接下来的目标。下面，我们正式开始填写代码，首先通过反射检查，我们传递的确实是一个函数。

```go
func Cacher(f interface{}, expiration time.Duration) (interface{}, error) {
	ft := reflect.TypeOf(f)
	if ft.Kind() != reflect.Func {
		return nil, errors.New("Only for functions")
	}
	return f, nil
}
```

接下来，创建一个结构体用来存放我们的输入参数。在创建结构体时，我们需要保证必须有一个输入和一个输出，并且所有的输入都是可比较的。

```go
unc buildInStruct(ft reflect.Type) (reflect.Type, error) {
	if ft.NumIn() == 0 {
		return nil, errors.New("Must have at least one param")
	}
	var sf []reflect.StructField
	for i := 0; i < ft.NumIn(); i++ {
		ct := ft.In(i)
		if !ct.Comparable() {
			return nil, fmt.Errorf("parameter %d of type %s and kind %v is not comparable", i+1, ct.Name(), ct.Kind())
		}
		sf = append(sf, reflect.StructField{
			Name: fmt.Sprintf("F%d", i),
			Type: ct,
		})
	}
	s := reflect.StructOf(sf)
	return s, nil
}

func Cacher(f interface{}, expiration time.Duration) (interface{}, error) {
	ft := reflect.TypeOf(f)
	if ft.Kind() != reflect.Func {
		return nil, errors.New("Only for functions")
	}

	inType, err := buildInStruct(ft)
	if err != nil {
		return nil, err
	}

	if ft.NumOut() == 0 {
		return nil, errors.New("Must have at least one returned value")
	}

	fmt.Println("inType looks like", inType)
	return f, nil
}
```

接下来，还剩最后一步，定义一个 map 变量，用它存放输入输出的缓存，并且用 reflection 反射在 f 函数基础上生成具有缓存能力的新函数。

```go
type outExp struct {
	out    []reflect.Value
	expiry time.Time
}

func Cacher(f interface{}, expiration time.Duration) (interface{}, error) {
	ft := reflect.TypeOf(f)
	if ft.Kind() != reflect.Func {
		return nil, errors.New("Only for functions")
	}

	inType, err := buildInStruct(ft)
	if err != nil {
		return nil, err
	}

	if ft.NumOut() == 0 {
		return nil, errors.New("Must have at least one returned value")
	}

	m := map[interface{}]outExp{}
	fv := reflect.ValueOf(f)
	cacher := reflect.MakeFunc(ft, func(args []reflect.Value) []reflect.Value {
		iv := reflect.New(inType).Elem()
		for k, v := range args {
			iv.Field(k).Set(v)
		}
		ivv := iv.Interface()
		ov, ok := m[ivv]
		now := time.Now()
		if !ok || ov.expiry.Before(now) {
			ov.out = fv.Call(args)
			ov.expiry = now.Add(expiration)
			m[ivv] = ov
		}
		return ov.out
	})
	return cacher.Interface(), nil
}
```

完成！

再来看一下上面的代码，首先，我们定义了一个结构体 outExp，用它存放输出和过期时间。

接着，我们定义了一个 map，它的 key 是interface{}，值是 outExp 类型。它们的选择都是有原因的。先说 key 是 interface{} 类型的原因。之前的例子，我们通过反射创建了一个结构体，这种结构体没有名称，为了存储实例，我们不得不使用 interface{} 类型表示。关于返回类型，当你用反射调用函数，它的返回类型是 []reflect.Value。同样，传递给 MakeFunc 的闭包函数返回的也是这种类型的值。 为了避免值拷贝，我们通过 []reflect.Value 保存返回值，并把它存入 map。

在闭包中，我们通过反射构造了一个自定义类型的实例，将传给函数的参数放入其中。接着，检查 m 中是否存在实例等于它，如果没有，或已经过期，我们将调用包裹函数，然后将响应结果和过期时间保存进变量 ov 中。接着，以自定义结构体的实例为 key，将 ov 保存进 m 中。最后，返回 ov.out 即可。

到此，我们正式完成了 Cacher 工厂函数，它可以包裹 Go 中几乎所有的函数，实现一定期限的缓存。

我们如何使用呢？一个例子，如下：

```
func AddSlowly(a, b int) int {
	time.Sleep(100 * time.Millisecond)
	return a + b
}

func main() {
	ch, err := Cacher(AddSlowly, 2*time.Second)
	if err != nil {
		panic(err)
	}
	chAddSlowly := ch.(func(int, int) int)
	for i := 0; i < 5; i++ {
		start := time.Now()
		result := chAddSlowly(1, 2)
		end := time.Now()
		fmt.Println("got result", result, "in", end.Sub(start))
	}
	time.Sleep(3 * time.Second)
	start := time.Now()
	result := chAddSlowly(1, 2)
	end := time.Now()
	fmt.Println("got result", result, "in", end.Sub(start))
}
```

例子中，仅仅是 sleep 100ms，然后，求两个数的和。实际的情况可能是，数据库的查询或网络服务的调用。由于 Go 没有泛型，Cacher 返回的函数需要转化为合适的类型，错误检查也要求有几行代码，interface{} 的 ch 也需要转化为实际的函数类型。

执行代码，将会得到如下的输出：

```go
$ go run cacher.go
got result 3 in 100.079405ms
got result 3 in 3.873µs
got result 3 in 561ns
got result 3 in 462ns
got result 3 in 398ns
got result 3 in 100.054602ms
```

第一次执行占用了大概 100 ms（计算处理时间），接下里的几次调用都只用了几百纳秒。在暂停了 3 秒后，执行最后一次，再次耗时 100 ms。

[运行示例](https://play.golang.org/p/GNXG4CpG-E)

# 新的秘密武器

再说最后一点，反射对性能有一定的影响。如果执行的是密集性运算，或是调用网络服务，通过反射在上面加一层代码，这对性能将不会有太大的影响。但是，多数代码都是非常快的。极有可能，你代码中的大部分方法的执行时间都是几百毫秒以内，此类场景，就要小心反射的使用了，此时，反射会性能有较大影响，我们需要重新思考是否值得。

总而言之，当我们再遇到各种类型的问题时，可以回想下反射提供的可能。当遇到看起不可能解决的问题时，比如虽然两个类型的处理逻辑相似，但是类型的自身的共性不多，这时候，反射将成为你的秘密武器。

我的博文：[Go 中如何使用反射 Part Two](https://www.poloxue.com/posts/2019-08-17-reflection-in-golang-part2)，译：[Learning to Use Go Reflection — Part 2](https://medium.com/capital-one-tech/learning-to-use-go-reflection-part-2-c91657395066)
