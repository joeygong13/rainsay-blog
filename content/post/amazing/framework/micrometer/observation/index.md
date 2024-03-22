---
title: "Micrometer Observation"
date: 2024-03-22T10:53:13+08:00
draft: false
description: "Micrometer provides a simple facade over the instrumentation clients for the most popular observability systems, allowing you to instrument your JVM-based application code without vendor lock-in. Think SLF4J, but for observability."
categories: ['奇妙的世界']
tags: ['monitor', 'java']
image: "image/micrometer.png"
---


## 1. Observation API

常用的监测服务的方式为记录日志、维护各种 Meter 数据，这些代码凌乱的散落在各个类中，是一种 low level 的 api。从 Micrometer 1.10 开始，Micrometer 提供了一种名为 `Observation API` 的更高层级的 API 来保持应用的可观察性。Maven 引入依赖

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-observation</artifactId>
    <version>1.11.4</version>
</dependency>

```

服务的可观测性，可抽象表示为了解一系列事件的发生情况，和事件的统计情况。Observation API 的设计围绕此展开的

## 2. ObservationRegistry

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

 每个 Observation 可以在 context 添加一些元数据，这些元数据可以有效的帮助我们进行 debug。
 官方文档有如下解释：

 > To make it possible to debug production problems an Observation needs additional metadata such as key-value pairs (also known as tags). You can then query your metrics or distributed tracing backend by those tags to find the required data. Tags can be either of high or low cardinality.
 
 基位代表着数据的离散程度，有很多不同的数据表示高基位，有限的不同数据为低基位

```java
// 创建一个事件监听类，它将会集中处理事件，并通过事件来记录各种 日志、metric。
static class SimpleHandler implements ObservationHandler<Observation.Context> {

    @Override
    public void onStart(Observation.Context context) { }

    @Override
    public void onError(Observation.Context context) { }

    @Override
    public void onEvent(Observation.Event event, Observation.Context context) { }

    @Override
    public void onStop(Observation.Context context) { }

    @Override
    public boolean supportsContext(Observation.Context handlerContext) { return true }
}

// 业务类
ObservationRegistry registry = ObservationRegistry.create();
registry.observationConfig().observationHandler(new SimpleHandler());
Observation.Context context = new Observation.Context().put("key", "value");
Observation observation = Observation.start("my.operation", () -> context, registry);
try (Observation.Scope scope = observation.openScope()) {
    // 真正的业务处理
    observation.lowCardinalityKeyValue("my.lastname", "hello")
    doSomeWork1();
    observation.event(Observation.Event.of("my.event", "look what happend"));
    doSomeWork2();
    observation.highCardinalityKeyValue("my.firstname", "rainsay")
}
catch (Exception ex) {
    observation.error(ex); // and don't forget to handle exceptions
    throw ex;
}
finally {
    observation.stop();
}

// 如果业务简单地，可以直接 observation.observa()
```

你可以通过继承 `Observation.Context` 来创建一个特定的上下文，以便 Handler 特殊处理。还可以通过创建 ObservationConvention 实现类来覆写默认的名称


