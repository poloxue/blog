---
title: "Go 中 struct tag 如何用？基于它实现字段级别的访问控制"
date: 2024-01-28T18:56:59+08:00
draft: false
comment: true
description: "在Go 中，结构体主要是用于定义复杂数据类型。而 struct tag 则是附加在 struct 字段后的字符串，提供了一种方式来存储关于字段的元信息。然后，tag 在程序运行时不会直接影响程序逻辑。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-26-struct-tag-uses-in-golang-01.png)

> 嗨，大家好！本文是系列文章 Go 小技巧第十篇，系列文章查看：[Go 语言小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=3291066778475053060)。

在 Go 中，结构体主要是用于定义复杂数据类型。而 struct tag 则是附加在 struct 字段后的字符串，提供了一种方式来存储关于字段的元信息。然后，tag 在程序运行时不会直接影响程序逻辑。

本文将会基于这个主题展开，讨论 Go 中的结构体 tag 究竟是什么？我们该如何利用它？另外，文末还提供了一个实际案例，帮助我们进一步提升对 struct tag 的理解。

## struct tag 是什么？

如下是一个定义了 tag 的结构体 Person 类型。

```go
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
```

例子中，`json:"name"`和`json:"age"` 就是结构体 tag。结构体 tag 的使用非常直观。你只需要在定义结构体字段后，通过反引号 `` 包裹起来的键值对形式就可定义它们。

## 具体有什么用呢？

这个 tag 究竟有什么用呢？为何要定义它们。

单从这个例子中来看，假设你是在 "encoding/json"  库中使用 `Person` 结构体，它是告诉 Go 在处理 JSON 序列化和反序列化时，字段名称的转化规则。

让我们通过它在 "encoding/json" 的使用说明它的效果吧。

```go
p := Person{Name: "John", Age: 30}
jsonData, err := json.Marshal(p)
if err != nil {
    log.Println(err)
}
fmt.Println(string(jsonData)) 
```

输出: 
```json
{"name":"John","age":30}
```

可以看到输出的 JSON 的 `key` 是 name 和 age，而非 Name 和 Age。

与其他语言对比的话，虽然 Go 的 struct tag 在某种程度上类似于 Java 的注解或 C# 的属性，但 Go 的 tag 更加简洁，并且主要通过反射机制在运行时被访问。

这种设计反映了Go语言的哲学：简单、直接而有效。但确实也是功能有限！

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-26-struct-tag-uses-in-golang-02-v2.png)

## 常见使用场景

结构体 tag 在 Go 语言中常见用途，我平时最常见有如下这些。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-26-struct-tag-uses-in-golang-05.png)

### JSON/XML 序列反序列化

如前面的介绍的案例中，通过 `encoding/json` 或者其他的库如 `encoding/xml` 库，tag 可以控制如何将结构体字段转换为 JSON 或 XML，或者如何从它们转换回来。

### 数据库操作

在ORM（对象关系映射）库中，tag 可以定义数据库表的列名、类型或其他特性。

如我们在使用 Gorm 时，会看到这样的定义：

```go
type User struct {
    gorm.Model
    Name   string `gorm:"type:varchar(100);unique_index"`
    Age    int    `gorm:"index:age"`
    Active bool   `gorm:"default:true"`
}
```

结构体 tag 可用于定义数据库表的列名、类型或其他特性。

### 数据验证

在一些库中，tag 用于验证数据，例如，确保一个字段是有效的电子邮件地址。

如下是 [govalidator](github.com/asaskevich/govalidator) 使用结构体上 tag 实现定义数据验证规则的一个案例。

```go
type User struct {
    Email string `valid:"email"`
    Age   int    `valid:"range(18|99)"`
}
```

在这个例子中，valid tag 定义了字段的验证规则，如 `email` 字段值是否是有效的 `email`，`age` 字段是否满足数值在 18 到 99 之间等。

我们只要将类型为 `User` 类型的变量交给 `govalidator`，它可以根据这些规则来验证数据，确保数据的正确性和有效性。

示例如下：

```go
valid, err := govalidator.ValidateStruct(User{Email: "test@example.com", Age: 20})
```
返回的 `valid:` 为 `true` 或 `false`，如果发生错误，`err` 提供具体的错误原因。

## tag 行为自定义

前面展示的都是利用标准库或三方库提供的能力，如果想自定义 tag 该如何实现？毕竟有些情况下，如果默认提供的 tag 提供的能力不满足需求，我们还是希望可以自定义 tag 的行为。

这需要了解与理解 Go 的反射机制，它为数据处理和元信息管理提供了强大的灵活性。

如下的示例代码：

```go
type Person struct {
    Name string `mytag:"MyName"`
}

t := reflect.TypeOf(Person{})
field, _ := t.FieldByName("Name")
fmt.Println(field.Tag.Get("mytag")) // 输出: MyName
```

在这个例子中，我们的 `Person` 的字段 `Name` 有一个自定义的 tag - `mytag`，我们直接通过反射就可以访问它。

这只是简单的演示如何访问到 tag。如何使用它呢？

这就要基于实际的场景了，当然，这通常也离不开与反射配合。下面我们来通过一个实际的例子介绍。

## 案例：结构体字段访问控制

让我们考虑一个实际的场景：一个结构访问控制系统。

这个系统中，我们可以根据用户的角色（如 admin、user）或者请求的来源（admin、web）控制对结构体字段的访问。具体而言，假设我定义了一个包含敏感信息的结构体，我可以使用自定义 tag 来标记每个字段的访问权限。

是不是想到，这或许可用在 API 接口范围字段的控制上，防止泄露敏感数据给用户。

接下来，具体看看如何做吧？

### 定义结构体

我们首先定义一个`UserProfile`结构体，其中包含用户的各种信息。每个信息字段都有一个自定义的 `access` tag，用于标识字段访问权限（`admin`或`user`）。

```go
type UserProfile struct {
    Username    string `access:"user"`  // 所有用户可见
    Email       string `access:"user"`  // 所有用户可见
    PhoneNumber string `access:"admin"` // 仅管理员可见
    Address     string `access:"admin"` // 仅管理员可见
}
```

其中，`PhoneNumber` 和 `Address` 是敏感字段，它只对 admin 角色可见。而 `UserName` 和 `Email` 则是所有用户可见。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-26-struct-tag-uses-in-golang-04-v1.png)

到此，结构体 `UserProfile` 定义完成。

### 实现权限控制

接下来就是要实现一个函数，实现根据 `UserProfile` 定义的 `access` tag 决定字段内容的可见性。

假设函数名称为 `FilterFieldsByRole`，它接受一个 `UserProfile` 类型变量和用户角色，返回内容一个过滤后的 `map`（由 fieldname 到 fieldvalue 组成的映射），其中只包含角色有权访问的字段。

```go
func FilterFieldsByRole(profile UserProfile, role string) map[string]string {
    result := make(map[string]string)
    val := reflect.ValueOf(profile)
    typ := val.Type()

    for i := 0; i < val.NumField(); i++ {
        field := typ.Field(i)
        accessTag := field.Tag.Get("access")
        if accessTag == "user" || accessTag == role {
            // 获取字段名称
            fieldName := strings.ToLower(field.Name) 
            // 获取字段值
            fieldValue := val.Field(i).String() 
            // 组织返回结果 result
            result[fieldName] = fieldValue
        }
    }
    return result
}
```

权限控制的重点逻辑部分，就是 `if accessTag == "user" || accessTag == role`  这段判断条件。当满足条件之后，接下来要做的就是通过反射获取字段名称和值，并组织目标的 Map 类变量 `result`了。

### 使用演示

让我们来使用下 `FilterFieldsByRole` 函数，检查下是否满足按角色访问特定的用户信息的功能。

示例代码如下：

```go
func main() {
    profile := UserProfile{
        Username:    "johndoe",
        Email:       "johndoe@example.com",
        PhoneNumber: "123-456-7890",
        Address:     "123 Elm St",
    }

    // 假设当前用户是普通用户
    userInfo := FilterFieldsByRole(profile, "user")
    fmt.Println(userInfo)

    // 假设当前用户是管理员
    adminInfo := FilterFieldsByRole(profile, "admin")
    fmt.Println(adminInfo)
}
```

输出：

```bash
map[username:johndoe email:johndoe@example.com]
map[username:johndoe email:johndoe@example.com phonenumber:123-456-7890 address:123 Elm St]
```

这个场景，通过自定义结构体 tag，给予指定角色，很轻松地就实现了一个基于角色的权限控制。

毫无疑问，这个代码更加清晰和可维护，而且具有极大灵活性、扩展性。如果想扩展更多角色，也是更加容易。

不过还是要说明下，如果在 API 层面使用这样的能力，还是要考虑反射可能带来的性能影响。

## 总结

这篇博文介绍了Go语言中结构体 tag 的基础知识，如是什么，如何使用。另外，还介绍了它们在不同场景下的应用。通过简单的例子和对比，我们看到了 Go 中结构体 tag 的作用。

文章的最后，通过一个实际案例，演示了如何使用 struct tag 使我们代码更加灵活强大。虽然 struct tag 的使用非常直观，但正确地利用这些 tag 可以极大提升我们程序的功能和效率。

博文地址：[Go 中 struct tag 如何用？基于它实现字段级别的访问控制](https://www.poloxue.com/posts/2024-01-26-struct-tag-uses-in-golang/)
