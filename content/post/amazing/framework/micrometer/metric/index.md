---
title: "Micrometer Metric"
description: "Micrometer provides a simple facade over the instrumentation clients for the most popular observability systems, allowing you to instrument your JVM-based application code without vendor lock-in. Think SLF4J, but for observability."
date: 2024-03-21T10:53:13+08:00
draft: false
categories: ['奇妙的世界']
tags: ['monitor', 'java']
image: "image/micrometer.png"
---

## 1. Registry

Meter 代表着应用的一个数据指标，如 http 请求数、CPU 占用率等。在 Micrometer 中，所有 Meter 数据都由 `MeterRegistry` 持有，MeterRegistry 的作用是管理 Meter 数据，并把他们导出到对应的数据平台。每个受支持的监管系统都有自己的 Registry 实现。如 prometheus 有自己的实现，maven 添加

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.11.4</version>
</dependency>

```

Micrometer 自身有一个名为 `SimpleMeterRegistry` 的 registry，它把所有数据存储在内存中，并且不导出数据。如果你还没有选好监测平台，你可以使用它来快速开始。

```java
MeterRegistry registry = new SimpleMeterRegistry();
```

**Composite Registries**

Micrometer 提供了 `CompositeMeterRegistry`，你可以添加多个 registry 到这个类中，以方便你把 Meter 数据导出到多个监测平台。

```java
CompositeMeterRegistry composite = new CompositeMeterRegistry();

Counter compositeCounter = composite.counter("counter");

// increment counter dont effect，it is still yields 0, because no registry be added to composite,
compositeCounter.increment();

composite.add(new SimpleMeterRegistry());

compositeCounter.increment();
```

{{< notice tip >}}

Micrometer 提供了一个全局 Registry，它是一个 Composite Registry。使用 `Metrics.globalRegistry` 来获取他的实例。Metrics 类有一系列创建 Meter 的静态方法，默认使用该全局 Registry 创建

{{< /notice >}}

## 2. Meters

Meter 是一种具名数据，代表着你想监控的某种数据，并且可以打上多种 Tag 来标记。Meter 有许多种类，如：

+ Counter,计数器，代表一种只能递增的数据，如 http 请求数，缓存命中数等
+ Gauge, 代表一种瞬时数据，如 CPU 使用率、内存占用率
+ TimeGuage，类似 gauge，只不过单位为时间
+ Timer, 计时器，代表短暂事件持续事件, 如 任务执行时间，它存储的是一系列的【事件-持续时间】，但展示的为聚合数据：count, max, min, mean，和时间水位线等
+ LongTaskTimer, 记录着仍在运行的任务的运行时间，普通 Timer 只有在任务结束时才被记录持续时间。它将至少展示这几种数据：活跃的任务数量，所有活跃任务的持续时间，所有活跃任务的最大持续时间。
+ FunctionCounter, counter 的特殊类型，你可以自定义 counter 的数据逻辑，这是因为有些监测系统接收全量 counter 数据，也有些只接收上次以来的增量数据。
+ FunctionTimer，同上
+ DistributionSummary, 分布汇总数据，可导出水位线数据


### 创建 Meter

```java
SimpleMeterRegistry r = new SimpleMeterRegistry();

// 1. 使用 register 创建
Counter counter = r.counter("http.request.count");

// 2. 使用 builer 创建
Counter errCounter = Counter.builder("http.request.count.error").register(r);
```

使用 Builder 模式还可以为每个 meter 添加 tag、数据单位、描述等详细信息，如

```java
Gauge.builder("world", () -> 1).baseUnit("kg").tags("tag1", "tag1value").description("World weight");
```

### Meter 添加 Tag 

tag 的作用类似于元数据，标记数据的额外信息

不仅可在创建 meter 时自定义添加，还可以统一在 registry 中添加，在 registry 中的 tag 一般代表通用属性，如 app name、location 等

```java
registry.config.commonTags("appName", "foo-app", "location", "China/Shanghai")
```

### Meter 的打印

使用 SimpleMeterRegistry 可以很方便的调试 Meter 数据，它有一个方法 `getMeterAsString()` 可以快速打印出所有托管的 Meter 数据。

PrometheusMeterRegistry 也有一个 `scrape()` 的方法用来打印 prometheus 风格的 Meter 数据

