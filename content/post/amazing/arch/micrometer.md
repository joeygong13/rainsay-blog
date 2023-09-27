---
title: "Micrometer"
description: "Micrometer provides a simple facade over the instrumentation clients for the most popular observability systems, allowing you to instrument your JVM-based application code without vendor lock-in. Think SLF4J, but for observability."
date: 2023-09-27T19:11:49+08:00
draft: false
categories: ['奇妙的世界']
tags: ['arch', 'monitor', 'java']
image: "micrometer.png"
---

## 1. Metric

### 1.1 Registry

Meter 代表着应用的一个数据指标，如 http 请求数、CPU 占用率等。在 Micrometer 中，所有 Meter 数据都有 `MeterRegistry` 持有，MeterRegistry 的作用是管理 Meter 数据，并把他们导出到对应的数据平台。每个受支持的监管系统都有自己的 Registry 实现。如 prometheus 有自己的实现，maven 添加

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
// counter dont effect，it is still yields 0, because no registry be added to composite,
compositeCounter.increment();

SimpleMeterRegistry simple = new SimpleMeterRegistry();
composite.add(simple);

compositeCounter.increment();
```

> Micrometer 提供了一个全局 Registry，它是一个 Composite Registry。使用 `Metrics.globalRegistry` 来获取他的实例。Metrics 类有一系列创建 Meter 的静态方法，默认使用该全局 Registry 创建

### 1.2 Meters

Meter 是一种具名数据，并且可以打上多种 Tag 来标记。Meter 有许多种类，如：

+ Counter,计数器，代表一种只能递增的数据，如 http 请求数，缓存命中数等
+ Gauge, 代表一种瞬时数据，如 CPU 使用率、内存占用率
+ TimeGuage，类似 gauge，只不过单位为时间
+ Timer, 计时器，代表短暂事件持续事件, 如 http 响应时间，它存储的是一系列的【事件-持续时间】，但展示的为聚合数据：count, max, min, mean，和时间水位线等
+ LongTaskTimer, 记录着仍在运行的任务的运行时间，普通 Timer 只有在任务结束时才被记录持续时间。它将至少展示这几种数据：活跃的任务数量，所有活跃任务的持续时间，所有活跃任务的最大持续时间。
+ FunctionCounter, counter 的特殊类型，你可以自定义 counter 的数据逻辑，这是因为有些检测系统接收全量 counter 数据，也有些只接收上次以来的增量数据。
+ FunctionTimer，同上
+ DistributionSummary, 分布汇总数据，可导出水位线数据


创建方法

```java
// 直接 registry 创建，并添加 tag
timer = registry.timer("http.server.respone", "uri", "/api/example")

// build 创建
timer = Timer.builder("my.timer").tag("uri", "/api/example").register(registry);
```

关于 tag，不仅可以每个 meter 自定义添加，还可以统一在 registry 中添加，在 registry 中的 tag 一般代表通用属性，如 app name、location 等

```java
registry.config.commonTags("appName", "foo-app", "location", "China/Shanghai")
```


## 2. Observation API

常用的监测服务的方式为记录日志、维护各种 Meter 数据，这些代码凌乱的散落在各个类中，是一种 low level 的 api。从 Micrometer 1.10 开始，Micrometer 提供了一种名为 `Observation API` 的更高层级的 API 来保持应用的可观察性。Maven 引入依赖

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-observation</artifactId>
    <version>1.11.4</version>
</dependency>

```

服务的可观测性，可抽象表示为了解一系列事件的发生情况，和事件的统计情况。Observation API 的设计围绕此展开的

### 2.1 ObservationRegistry

obervation api 的核心类

### 2.1 Observation Handler

在以前的开发中，我们通常手动处理每个 Meter 数据，如检测某个方法的耗时情况

```java
MeterRegistry registry = new SimpleMeterRegistry();
Timer.Sample sample = Timer.start(registry);
try {
    // do some work here
}
finally {
    sample.stop(Timer.builder("my.timer").register(registry));
}
```

这对单个 meter 的处理，看起来还算美妙，但如果需要在此代码中添加更多的监控数据，可能会增加更多的代码量，这将导致代码的可维护性和可读性降低。Oberrvation API 就是来解决此问题的，它使用了类似数据总线的思维，通过传递事件和上下文来集中记录数据。请看它的解决方案：

通过监听以下生命周期：

+ `start` - Observation has been started. Happens when the Observation#start() method gets called.
+ `stop` - Observation has been stopped. Happens when the Observation#stop() method gets called.
+ `error` - An error occurred while observing. Happens when the Observation#error(exception) method gets called.
+ `event` - An event happened when observing. Happens when the Observation#event(event) method gets called.
+ `scope started` - Observation opens a Scope. The Scope must be closed when no longer used. Handlers can create thread local variables on start that are cleared upon closing of the scope. Happens when the Observation#openScope() method gets called.
+ `scope stopped` - Observation stops a Scope. Happens when the Observation.Scope#close() method gets called.

> 每个 Observation 有 一些元数据，暂时不知道有什么用，官方文档有如下解释：
> To make it possible to debug production problems an Observation needs additional metadata such as key-value pairs (also known as tags). You can then query your metrics or distributed tracing backend by those tags to find the required data. Tags can be either of high or low cardinality.
> 基位代表着数据的离散程度，有很多不同的数据表示高基位，有限的不同数据为低基位

```java
// 创建一个事件监听类，它将会集中处理事件，并通过事件来记录各种 日志、metric。
static class SimpleHandler implements ObservationHandler<Observation.Context> {

    @Override
    public void onStart(Observation.Context context) {
        // 采样开始
        System.out.println("START " + "data: " + context.get("my.name"));
    }

    @Override
    public void onError(Observation.Context context) {
        // 在采样中，有异常发生
        System.out.println("ERROR " + "data: " + context.get("my.name") + ", error: " + context.getError());
    }

    @Override
    public void onEvent(Observation.Event event, Observation.Context context) {
        // 在采样中，某事件发生了，如一个远程调用、一个数据库修改等
        System.out.println("EVENT " + "event: " + event + " data: " + context.get("my.name"));
    }

    @Override
    public void onStop(Observation.Context context) {
        // 采样结束
        System.out.println("STOP  " + "data: " + context.get("my.name"));
    }

    @Override
    public boolean supportsContext(Observation.Context handlerContext) {
        // 判断是否支持该事件上下文，如支付方法需要特定的事件处理器，普通处理器可以忽略
        return true;
    }
}
// 业务类

ObservationRegistry registry = ObservationRegistry.create();
registry.observationConfig().observationHandler(new SimpleHandler());
Observation.Context context = new Observation.Context().put("my.name", "test").put("my.id", 1);
Observation observation = Observation.start("my.operation", () -> context, registry);
try (Observation.Scope scope = observation.openScope()) {
    // 真正的业务处理
    doSomeWork1();
    observation.event(Observation.Event.of("my.event", "look what happend"));
    doSomeWork2();
}
catch (Exception ex) {
    observation.error(ex); // and don't forget to handle exceptions
    throw ex;
}
finally {
    observation.stop();
}

```

你可以通过继承 `Observation.Context` 来创建一个特定的上下文，以便 Handler 特殊处理。还可以通过创建 ObservationConvention 实现类来覆写默认的名称
