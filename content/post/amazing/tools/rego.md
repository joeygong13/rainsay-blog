---
title: "Rego 语法基础"
date: 2024-01-10T20:11:09+08:00
draft: true

categories: ['奇妙的世界']
tags: ['脚本', '决策', '语言']

---

## 1. 基础

## 1.1 语法

### 定义变量

```rego
# 基础类型
v := 1;
s := "hello"

# 对象
o := {"hello": "world"}

# 数组
a := ["hello", "world"]

# 集合
a1 := {"hello", "world", "world"} # will only one world element


```

### 数组操作

```rego

# 取第一个值
val := s1[0]

# 取第一个值
0, val in s1


```


## 1.2 迭代

普通迭代

```
val := s[_]

val := s[i]

some val in arr # 迭代值

some i, val in arr # 迭代索引和值

some i, _ in arr # 同上，省略值
```

`some` iterate，通过变量，自动把符合条件的索引存储在对应变量中

```rego
some i;
input.networks[i].public == true
```

Now the query asks for values of i that make the overall expression true. When you substitute variables in references, OPA automatically finds variable assignments that satisfy all of the expressions in the query. Just like intermediate variables, OPA returns the values of the variables.

You can substitute as many variables as you want. For example, to find out if any servers expose the insecure "http" protocol you could write:

```rego
some i, j;
input.servers[i].protocols[j] == 'http';
```


If variables appear multiple times the assignments satisfy all of the expressions. For example, to find the ids of ports connected to public networks, you could write:

```rego
some i, j
id := input.ports[i].id
input.ports[i].network == input.networks[j].id
input.networks[j].public
```

对于出现多次的 some 变量，会上下联动（即符合所有表达式的值将会提取，不管表达式顺序）

## 1.3 Rule

```rego
# 常量

a := {1,2,3}
b := {4,5,6}
c :=  a|b

# 条件判断

p := ture {

}

# 简写

p if {

}

# 赋值其他类型的条件判断

p := 5 if {

}

# 顺序规则

default a:=1
a:=5 if {

}else :=10 {
    
}

```

## 1.4 增量集合

```
# a_set will contain values of x and values of y
a_set[x] {...}
a_set[y] {...}

# alternatively above

a_set contains x if {...}
a_set contains y if {...}

```

上面的 x y 要在 {} 代码块中定义

## 1.5 Function

```rego
# 同规则

f(x, y) := true {

}

# 简写成 
f(x, y) if {

}

f(x) := "A" if {
    
}
```
