---
title: "Open Policy Agent - Rego 语言"
date: 2024-01-16T15:21:40+08:00
draft: false
categories: ["奇妙的世界"]
tags: ['脚本', 'OPA']
image: "image/opa-horizontal-color.png"
description: "如何使用 Rego 语言描述策略"
---

{{< mark >}} Rego 所有语句都是规则 Policy。{{< /mark >}}

## 1. 变量

Rego 中的变量定义称为 Constant Policy，类型有数字、字符串、数组、集合、对象、布尔、null等类型。

```rego
package repl

name := "tom"

num := 1

b := true

obj := {
    "name": "tom",
    "num": 1
}

arr := ["1", "2", "3", "3"]

set := {"1", "2", "3", "3"}
```

### 引用

类似 C 语言，Rego 使用 `.` dot 符号引用对象属性，使用方括号 `[]` 可以引用数组元素，也可以引用对象属性，如 arr[0], obj["name"]

当 OPA 进行策略计算时，使用**全局变量** `input` 引用输入数据，如 input.http_method，

**全局变量** `data` 存储了外部 data 数据，同时所有 rego 规则产生的值也存储在 data 上面。不过规则结果值的引用使用了 包名 进行隔离。

```rego
package repl
import rego.v1

set := {1,2,3,4}

is_joey if data.user == "joey"
```

可使用全路径名称 data.repl.set 对上面的 set 进行读取，其中 repl 为规则的包名。

而对于输入的 data.json 数据， 如下：

```json
{
    "user": "joey"
}
```

在 rego 文件中则可以直接使用 data.user 读取。

现在我们获取最顶级的 data 对象看看效果 opa eval --bundle . data，输出如下

{{< figure src="rego-out-00.png" alt="Rego 输出" title="Rego 输出" width=600 class="img-center" >}}

注意看：rego 脚本中的输出被包名隔离了，而 data.json 文件的数据则没有。

## 2. 表达式 Expression

为了产生规则的输出，需要表达式来对数据和输入进行判断。在同一个查询块中，多个表达式的关系是 AND 的关系，只有这个规则块中的所有表达式为 true 或有值时（表达式的顺序不影响结果），这个规则才有效，可以产生输出。

如果存在不为 true 或没有值的表达式，则规则结果为 undefined，注意不是 false，这是因为 false 可以作为结果输出，但 undefined 作为结果输出时是无效的。

{{< notice tip >}}
想要输出 false 可以使用 `default` 关键字定义规则的默认值。意为：当规则的结果为 undefined 时，将使用默认值代替。
{{< /notice >}}

```json
{
    "pets": [
        { "name": "tom", "type": "cat", "food": ["milk", "beef", "cheese"]},
        { "name": "jerry", "type": "mouse", "food": ["milk", "cheese"] },
        { "name": "spark", "type": "dog", "food": ["milk", "beef"]},
        { "name": "tak", "type": "dog", "food": ["milk"]},
    ]
}
```

下面表达式中第一个结果为 ture, 第二个结果为 undefined

```rego
package repl
# true
is_tom_cat if {
    s := input.pets[0]
    s.name == "tom"
    s.type == "cat"
}

# undefined 
is_tom_mouse if {
    s := input.pets[0]
    s.name == "tom"
    s.type == "mouse"
}

# false 
default is_tom_mouse_d := false
is_tom_mouse_d if {
    s := input.pets[0]
    s.name == "tom"
    s.type == "mouse"
}

```

### 迭代表达式

在 OPA 的迭代表达式中，有点类似 SQL 查询语句的味道了

#### `SOME` 迭代

表达式在判断逻辑时，同时会自动找出使表达式为 true 的变量值，同变量名在不同表达式中，则如果所有表达式为 true 的话，则会产生类似查询交集的效果。如我们要找到吃 beef 的狗狗

```rego
package repl
#    "beef_dog": [
#        "spark"
#    ]

beef_dog contains t if {
    some i
    users := input.users
    t := users[i].type
    users[i].food[_] == "beef"
    users[i].type == "dog"
}
```

看到了吗~~~，{{< mark >}} some 表示 i 是个多值类型，通过一步步表达式的过滤，把符合所有表达式为 true 的变量值存储在 i 中，这期间表达式的顺序不影响最终结果。同时变量 t 也自动与 i 进行绑定。{{< /mark >}}

多么神奇！！！ 更进一步，可以使用多个 some 变量，来组合过滤，如 i，j，k，最终 ijk 的值是对应的。

#### `ALL` 迭代

对于 ALL 迭代，不像 SOME，它要求所有元素都符合表达式，所以他没有类似查询的效果。例如是否所有宠物都不喝牛奶，或者不吃草

```rego
package repl
# true 
no_one_grass if {
    every pet in input.pets {
        not "grass" in pet.food
    }
}

# undefined
no_one_milk if {
    every pet in input.pets {
        not "milk" in pet.food
    }
}
```

## 3. 规则 Rule

规则由表达式组成。可以内嵌重复使用。可以有同名规则，同名规则之间是 {{< mark >}} 或 OR {{< /mark >}} 的关系。

Rego 文件中的规则名称，可以作为计算目标，他们的值存储在 `data` 变量中，总是可以使用 `data.<package-path>.<rule-name>` 来读取。

OPA 定义了两种类型规则，产生单个值的称为 Complete Rules；产生集合值的称为 Partial Rules。

### Complete Rules

Complete rules are if-then statements that assign a single value to a variable.

规则定义格式:  

```rego
<rule_name> := <value> if {
    expression...
}
```

样例, 对于 bool 值规则 `:= <value>` 可以省略

```rego
# true
has_tom := true if {
    some pet in input.pets 
    net.name == "tome"
}
```

### Partial Rules

Partial rules are if-then statements that generate a set of values and assign that set to a variable.

```rego
<rule_name> contains <var_name_in_query> if {
    expression...
    <var_name_in_query>
    expression...
}

```

例如：

```rego
#"eat_milk_pet": ["jerry", "spark", "tak", "tom" ],

eat_milk_pet contains pet.name if {
    some pet in input.pets
    pet.food[_] == "milk"
}
```

## 4. 函数

函数类似与规则，但函数只能返回单值，同名函数之间是或的关系，从上到下依次执行，知道函数里面的表达式全为真。

```rego
# with `import rego.v1` or `import future.keywords.if`
<fun_name>(<param>...) := <value> if {
    expression...
}
```

赋值函数

```rego
# with `import rego.v1` or `import future.keywords.if`
f(x) := "A" if { x >= 90 }
f(x) := "B" if { x >= 80; x < 90 }
f(x) := "C" if { x >= 70; x < 80 }
```


### 内置函数


常用函数列表，更多见 [内置函数大全](https://www.openpolicyagent.org/docs/latest/policy-reference/#built-in-functions)

| 用途 | 函数 |
|------|-----|
| 比较 | ==, >, <, >=, <=, != |
| 数字 | abs, ceil, floor, round, +, -, *, /, %, numbers.range, numbers.range_step |
| 聚合 | count, max, min, sum, sort |
| 数组 | array.concat, array.slice, array.reverse |
| 集合 | &, -, |, union, intersection |
| 字符 | concat, contains, endswith, startswith, split, substring, indexof, lower, upper, replace, trim|
