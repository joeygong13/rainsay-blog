---
title: "Open Policy Agent - 基础"
date: 2024-01-10T20:11:09+08:00
draft: false

categories: ['奇妙的世界']
tags: ['脚本', 'OPA']
image: "image/opa-horizontal-color.png"
description: "Open Policy Agent 工作流程"
---

## 1. 用途

{{< figure src="../opa-service.svg" alt="OPA 工作流程" title="OPA 工作流程" width=400 class="img-center" >}}

Open Policy Agent 简称 OPA，是一个策略系统，通过定义一些规则，然后输入数据匹配规则，以生成最终结果。

### 流程描述

上图最上面的 `Service` 代表着需要进行决策的系统，如：业务系统、CI/CD 系统、ElasticSearch、Kubernetes 等。`Service` 与 `OPA` 交互，OPA 作为策略计算系统并最终返回决策结果。

`Policy` 是一系列规则的集合，它使用 Rego 语言编写。`Data` 是预先定义好的数据，它是输出决策的重要依据，用来支撑规则的计算。 {{< mark >}}Policy 是决策逻辑，Data 是知识，Query 是查询，Decision 是决策结果 {{< /mark >}}

举个栗子：

一般业务系统都使用基于角色的权限规则（RBAC），通过定义权限字符并关联 URL，然后又根据现实中的角色组合权限，使不同角色可以使用不同的功能。使用 OPA 来描述，即：

1. 角色与权限是为上图中的 Data
2. 当前用户，及其操作动作，使用 Query 描述，这个 Query 可以自由定义，只要你的规则能正确读取即可
3. Policy 计算判断动作是否符合该用户权限
4. 最终产生该用户是否可以操作的决策结果

## 2. 使用

通过点击下载 [OPA](https://github.com/open-policy-agent/opa/releases) 程序，它是一个可执行文件，可直接运行。

```sh
opa --help

# 交互式命令行模式
opa run

# 以独立 http 模式运行
opa run -s

```

或者直接使用命令行来计算结果

```sh

# current dir has:
# 1. role.json, this is data file, use data.xxx ref var in rego file
# 2. policy.rego, this is policy file
# 3. input.json
# last string is eval target, prefix must be data, and policy is pacage, allow is packate rule
opa eval --data policy.rego --data role.json --input input.json 'data.policy.allow'

```

`data.policy.allow` 是执行目标，其中 data 是固定前缀，policy 是 policy 的包名，allow 是 rego 文件里面的规则名

由于在 opa 程序里面，rego 文件和 data 文件都是属于 `data` 变量下，{{< mark >}}所以我们定义规则名和数据顶级变量名时最好不要同名 {{< /mark >}}

### 样例文件

policy.rego file

```rego
package policy

allow {
    some u
    data.user[u] == "Admin"
    input.name == u
}
```

data.rego file

```json
{
    "user": {
        "joey": "tom"
    }
}
```

input.json file

```json
{
    "tom": "Admin"
}
```
