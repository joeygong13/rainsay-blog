---
title: "常用命令"
description: "一些常用命令速记"
date: 2023-08-09T16:36:47+08:00
draft: false
categories: ['奇妙的世界']
tags: ['bash', 'command']
image: "commands.png"
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

> find -exec 后面跟命令，需使用 ; 分号表示结束， {} 表示查找到的文件名

