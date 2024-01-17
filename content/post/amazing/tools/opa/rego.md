---
title: "Open Policy Agent - Rego 语言"
date: 2024-01-16T15:21:40+08:00
draft: true
categories: ["奇妙的世界"]
tags: ['脚本', 'OPA']
image: "image/opa-horizontal-color.png"
description: "如何使用 Rego 语言描述策略"
---

{{< mark >}} Rego 所有语句都是规则 Policy。{{< /mark >}}

## 1. 变量

Rego 中的变量定义称为 Constant Policy。

```opa
# string
name := "tom"

# number
num := 1

# bool
b := true

# object
obj := {
    "name": "tom",
    "num": 1
}

# array 1、2、3、3
arr := ["1", "2", "3", "3"]

# set 1、2、3
set := {"1", "2", "3", "3"}
```

### 引用

类似 C 语言，Rego 使用 `.` dot 符号引用对象属性，使用方括号 `[]` 可以引用数组元素，也可以引用对象属性，如 arr[0], obj["name"]

当 OPA 进行策略计算时，使用**全局变量** `input` 引用输入数据，如 input.http_method，**全局变量** `data` 有点特殊，他可以引用所有策略。上面讲到 Rego 中的变量定义可以理解为一种常量策略，而 Data 知识数据其实本质就是写在非 Rego 脚本文件中的 Rego 变量，所以它也是可以用 `data` 引用。来个栗子，下面是一个 rego 脚本

```opa
package repl

set := {1,2,3,4}
```

可使用全路径名称 data.repl.set 对上面的 set 进行读取，其中 repl 为规则的包名，在同一个包名下，可省略包名前面的路径直接使用 set 引用。 而对于输入的 data.json 数据， 如下：

```json
{
    "user": "joey"
}
```

{{< mark >}}在 rego 文件中则可以直接使用 data.user 读取。同时也可作为执行目标使用，只不过需要写为 data.Data.user {{< /mark >}}

## 2. 规则

Rego 文件中的{{< mark >}}规则名称，可以作为计算目标，并作为最终结果作为输出。{{< /mark >}}