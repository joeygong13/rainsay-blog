---
title: "文件系统的抽象"
date: 2024-01-20T16:22:17+08:00
draft: true
categories: ["奇妙的世界"]
tags: ["Linux", "文件系统"]
description: "Linux 文件系统，"
---

{{< bilibili BV1jM411W7jV >}}

## 1. 文件接口

+ open/close
+ read/write/lseek
+ fstat/ftruncate
+ unlink/mkdir/dup

{{< notice info >}}

`unlink` 意为解除 link，当一个文件的引用计数变为 0 时即为真正删除，Unix 文件系统中没有 delete 文件的系统调用

{{< /notice >}}

## 2. 文件系统抽象

Unix 系统将文件的相关信息和文件本身这两个概念加以区分，例如访问控制权限、大小、拥有者、时间等文件信息，被称为文件元数据，被存储到一个单独的数据结构中，被称为 `inode`，inode 是一个 32 位整数，他在同一个文件系统中是不同的，在不同文件系统中可以相同。所有文件系统都需要实现 inode，不管是磁盘文件系统还是内存文件系统。

如何找到 inode，通过文件名找到 inode id，再通过 inode id 找到 inode。

## 3. 目录结构

EXT2: linkedlist

目录存储着文件的引用，它是一个线性列表，所以当一个目录的文件数量很多时，删除和创建文件是一个很耗时的操作。

EXT3, EXT4: hashmap

目录碎片，随着目录下文件的增多，一个磁盘块存储不下，那么就需要更多磁盘块，但后续的磁盘块可能离上一个磁盘块很远，导致磁盘读取速度变慢。

目录的块不会自动回收

```mermaid
mindmap
    root[目录]
        大小
        数据结构
```
