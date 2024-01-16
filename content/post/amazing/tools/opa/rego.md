---
title: "Open Policy Agent - Rego 语言"
date: 2024-01-16T15:21:40+08:00
draft: true
categories: ["奇妙的世界"]
tags: ['脚本', 'OPA']
image: "image/opa-horizontal-color.png"
description: "Open Policy Agent 工作流程"
---

{{< mark >}} Rego 所有语句都是规则 Policy。{{< /mark >}}

## 1. 变量

Rego 中的变量定义称为 Constant Policy。

```opa
# string
name := "tom"

# number
num := 1

# object
obj := {
    name: "tom",
    num: 1
}

# array 1、2、3、3
arr := ["1", "2", "3", "3"]

# set 1、2、3
set := {"1", "2", "3", "3"}


```

## 2. 规则

