---
title: "常用命令"
description: "一些常用命令速记"
date: 2023-08-09T16:36:47+08:00
draft: false
categories: ['奇妙的世界']
tags: ['bash', 'command']
image: "image/commands.png"
math: false
---

## Java

### JVM

| Command | Note |
|---------|------|
| jmap -heap | Print java heap summary |
| jmap -dump:format=b,file=<outfile> | dump java heap |
| jstat -gcutil | Displays a summary about garbage collection statistics. |


### Maven

| Command | Note |
|---------|------|
| mvn dependency:tree | View MVN Dependency Tree |

### Bash

| Command | Note |
|---------|------|
| find . -type f -exec grep -l xxx {} \\; | 关键字搜索当前目录所有文件 |
| netstat -ntp \| awk '{if (\$NF == "12345" &amp;&amp; \$6 == "CLOSE_WAIT") print \$0}' \| wc -l | 统计某进程 CLOSE_WAIT TCP 数量 |

{{< notice info>}}
find -exec 后面跟命令，需使用 ; 分号表示结束， {} 表示查找到的文件名
{{< /notice >}}

