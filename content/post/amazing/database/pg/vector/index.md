---
title: "PostgreSQL 向量存储"
date: 2025-02-21T11:46:09+08:00
draft: false
description: "使用 PGVector 进行向量存储和相似搜索"
tags: ["数据库", "向量搜索"]
categories: ["奇妙的世界"]
---

## PGVector Extension

PostgreSQL 的 PGVector 扩展主要用于支持向量数据类型的存储和操作，以下是其主要功能：

向量类型支持：PGVector 添加了 vector 数据类型，允许在表中存储一维浮点数数组。这对于机器学习模型的嵌入、图像特征向量等非常有用。

相似度搜索：提供了基于距离度量（如余弦相似度、欧几里得距离）的索引和查询功能，使得可以高效地进行最近邻搜索或相似性匹配。

性能优化：利用 PostgreSQL 的 GiST 或 IVFFlat 索引来加速大规模向量数据集上的相似度搜索操作。

集成简便：作为 PostgreSQL 的一个扩展，安装简单，并且可以直接使用 SQL 语句进行向量相关的 CRUD 操作，无需额外的应用层逻辑处理。

通过这些特性，PGVector 为需要处理高维空间数据的应用程序提供了一个强大的工具，特别是在推荐系统、搜索引擎和个人化内容分发等领域。


## 安装

### 安装

可以源码安装，或直接使用包管理器安装，这里以 Ubuntu 举例：

```bash
apt install postgresql-16-pgvector
```

### 加载扩展

登录对应的数据库，如 `hello` 数据库需要使用向量搜索功能 

```bash
psql -d hello

# psql 客户端
CREATE EXTENSION vector;
```

## 使用

PGVector 支持的数据类型：

- vector: 一维浮点数数组, 主要使用类型，可以存储长达 2000 维的序列
- halfvec: 可以存储半可以存储长达 4000 维的序列
- bit: 二进制类型向量，可达 64000 维
- sparsevec：1000 维非零向量

PGVector 支持的索引类型：

- IVFFlat: 基于聚簇的向量索引，将向量空间划分维多个簇，每个簇由一个中心点表示。在查询时，先找到最接近的簇中心，然后在该簇内进行精确搜索。由于只搜索部分簇，可能会牺牲一定的精度，占用内存较小，构建时需要聚类过程，构建时间较长。适用于静态数据
- HNSW: 基于多层图结构向量索引，在高维空间中表现优异，查询速度通常比 IVFFlat 更快。通过多层图结构，可以在保证较高精度的同时实现快速搜索。构建时间较短，适合动态更新的数据集。内存占用高。适用于动态更新的数据集

查询语句的运算符，都是相似度查询，但使用的算法不一样：

- `<->` L2 距离
- `<+>` L1 距离
- `#`   内积距离
- `<=>` cos 距离
- `<~>` 哈曼距离
- `<%>` Jaccard 距离

### 示例

```sql

- 创建表
create table t_embedding (
    id serial primary key,
    vector vector(3),
    content text
)

- 插入向量

insert into t_embedding (vector, content) values ("[1,2,3]", "123")

- 查询

select * from t_embedding where vector <-> "[2,2,3]" limit 1

- 创建索引
- 按照使用的向量数据类型和匹配算法来选择索引
- vector 可选 vector_l2_ops, vector_cos_ops, vector_pd_ops ... halfvec_l2_ops (其他数据类型，只需改变前缀即可)

create index idx_embedding_vector on t_embedding using hnsw (vector vector_l2_ops) with (m = 16, ef_construction = 64)
```
IVFFlat 类型索引，包含一个可选参数
- lists, 指定簇的多少 如果查询速度是主要关注点，可以适当减小 lists 值，但可能会降低搜索精度。如果搜索精度是主要关注点，可以适当增大 lists 值，但可能会增加查询时间。

HNSW 类型索引，包含两个可选参数
- m - the max number of connections per layer (16 by default)
- ef_construction - the size of the dynamic candidate list for constructing the graph (64 by default)