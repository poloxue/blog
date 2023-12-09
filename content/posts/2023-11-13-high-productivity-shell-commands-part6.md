---
title: "我的终端环境：高效 shell 命令（六）- JSON 神器 jq 命令"
date: 2024-02-12T15:40:41+08:00
draft: true
comment: true
description: "本文介绍一个更符合现代风格的 Unix 命令的集合"
---

本文将介绍一个命令 - [jq](https://github.com/jqlang/jq)，前文已经提过，一款可用于处理解析 JSON 文本的命令，它非常强大，甚至是可编程的。

# 案例

最简单的使用场景，JSON 文本格式化，命令如下所示：

```
curl https://coderwall.com/bobwilliams.json | jq '.'
```

输出内容如下：

```
{
  "id": 26098,
  "username": "bobwilliams",
  "name": "Bob Williams",
  "location": "Charleston, SC",
  "karma": 1010,
  "accounts": {
    "github": "bobwilliams",
    "twitter": "_bobwilliams"
  },
  "about": "hacker, lifter, drummer and happily married, proud father of three\n\n\n[LinkedIn](http://www.linkedin.com/pub/bob-williams/17/a6a/314)\n\n",
  "title": "CTO",
  "company": "SPARC",
  "team": 4521,
  "thumbnail": "https://coderwall-assets-0.s3.amazonaws.com/uploads/user/avatar/26098/photo.JPG",
  "endorsements": 1010,
  "specialities": [],
  "badges": [
    {
      "name": "Platypus",
      "description": "Have at least one original repo where scala is the dominant language",
      "created_at": "2014-04-22T09:02:31.443Z",
      "badge": "https://dj1symwmxvldi.cloudfront.net/assets/badges/platypus-edcb8d16952f9cb27e9a1644d7d38b5606ff8f0c8a55869c4e0d8c42ed4ea637.png"
    },
    { ... }, { ... }, { ... }, { ... }, { ... },
  ]
}
```

如果是通过 httpie 发起请求，jq 的这个能力就显的有点鸡肋了，虽说文本来源不一定都是 HTTP，但它还是占了大头不是。

jq 的强大之处，它可以访问 JSON 其中的部分信息，如 object 字段的部分访问，数组访问，使用函数统计、排序、过滤等。

我以一个简单的例子，将以上 JSON 处理为如下形式：

```json
{
  "name": "xxx",
  "title": "xxx",
  "company": "xxx",
  "badges": ["xxx01", "xxx02", ...]
}
```

如 object 类型，可展示部分字段。

如访问字段：

```
https ://coderwall.com/bobwilliams.json | jq '.name' # 访问字段
```

输出内容：

```bash
"Bob Williams"
```

访问字段（object）的字段

```bash
https ://coderwall.com/bobwilliams.json | jq '.accounts.github'
```

输入内容：

```bash
"bobwilliams"
```

选择部分字段，重新构造 object：

```bash
https ://coderwall.com/bobwilliams.json | jq '{name: .name, title: .title, company: .company}'
```

输出内容：

```bash
{
  "name": "Bob Williams",
  "title": "CTO",
  "company": "SPARC"
}
```

或者简化写法：

```bash
https ://coderwall.com/bobwilliams.json | jq '{name, title, company}'
```

输出与上面相同。

jq 支持复杂的数组访问，其中的 .badge 是一个数组：

```
https ://coderwall.com/bobwilliams.json | jq '.badges'
```

可基于下标访问：

```bash
https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1]'
```

输出内容：

```bash
{
  "name": "Python",
  "description": "Would you expect anything less? Have at least one original repo where Python is the dominant language",
  "created_at": "2013-09-30T20:27:44.031Z",
  "badge": "https://dj1symwmxvldi.cloudfront.net/assets/badges/python-df746136c133c10a796100239e3563e3419fb4d5a7b8cc9bed181c941f6dbf55.png"
}
```

或者是切片操作：

```bash
https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1:3]'
```

输出内容：

```bash
[
  {
    "name": "Python",
    "description": "Would you expect anything less? Have at least one original repo where Python is the dominant language",
    "created_at": "2013-09-30T20:27:44.031Z",
    "badge": "https://dj1symwmxvldi.cloudfront.net/assets/badges/python-df746136c133c10a796100239e3563e3419fb4d5a7b8cc9bed181c941f6dbf55.png"
  },
  {
    "name": "Narwhal",
    "description": "Have at least one original repo where Clojure is the dominant language",
    "created_at": "2013-09-30T20:27:43.755Z",
    "badge": "https://dj1symwmxvldi.cloudfront.net/assets/badges/narwhal-35d1b215da3a866b69fa1a7ffdd6d495c2b1f982fa65c7bba167f4fd4289fae7.png"
  }
]
```

或者数据中的元素的字段信息：

```
https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1:3][].name'
```


  - array： 
    - https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[]'
    - https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1]'
    - https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1:3]'
    - https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1:3][].name'
    - https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '.[1:3][].name'
    - https ://coderwall.com/bobwilliams.json | jq '.badges' | .[1:3][].name'
    - https ://coderwall.com/bobwilliams.json | jq '.badges[]'
    - constructor 构造 array；
      - https ://coderwall.com/bobwilliams.json | jq '.badges' | jq '[.[1:3][].name]'
  - function
    - sort: 
      - sort: https ://coderwall.com/bobwilliams.json | jq '[.badges[].name]' | jq 'sort'
      - sort by object attribute: 
    - reverse
      - https ://coderwall.com/bobwilliams.json | jq '[.badges[].name]' | jq 'sort | reverse'
    - length: 
      - https ://coderwall.com/bobwilliams.json | jq '.badges[].name' | jq 'length'
      - https ://coderwall.com/bobwilliams.json | jq '.badges[].name | length'
  - 函数式编程：map reduce select
    - map:
      - https ://coderwall.com/bobwilliams.json | jq '.badges'  | jq 'map( {name: .name, name_length: .name | length} )' | jq 'map(select(.name_length > 6))'
    - select
      - https ://coderwall.com/bobwilliams.json | jq '.badges'  | jq 'map( {name: .name, name_length: .name | length} )' | jq 'map(select(.name_length > 6))'
    - reduce:
      - https ://coderwall.com/bobwilliams.json | jq '.badges | map( {name: .name, name_length: .name | length} )' | jq 'reduce .[] as $o(0; .+$o.name_length)'
