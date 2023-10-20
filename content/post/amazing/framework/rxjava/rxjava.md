---
title: "Rxjava"
date: 2023-10-20T17:03:24+08:00
draft: true
categories: ["奇妙的世界"]
tags: ["java", "响应式框架", "观察者", "设计模式"]
---

##  1. RxJava 基础

> 推荐一个的 Rx 代码可视化网站，用于理解各种操作符 https://rxviz.com/

### 1.1 Observable 

Observable 是 Rx 中的核心抽象，代表一个可观察的对象。其产生数据流，并可以结合各种操作符来改变数据流。

常用的创建 Observable 方法有两种：

1. 使用 fromXxx(), just() 等方法从现有对象中创建;
2. 使用 Observable#create() 来自定义数据流；

```java
Observable<Integer> just = Observable.just(10);
Observable<Integer> integerObservable = Observable.fromArray(1, 2, 3, 4, 5);
Observable<String> stringObservable = Observable.<String>create(c -> {
    c.onNext("hello");
    c.onNext("world");
    c.onNext("this");
    c.onComplete();
});
```

create() 方法的参数为 ObservableOnSubscribe 类型，是一个以 Emitter 为参数的函数。当有订阅者订阅 Observable 时，ObservableOnSubscribe 表示的方法便会执行一次，以产生数据。

考虑到在 ObservableOnSubscribe 中，如上 `c.onNext("hello")`, 便是直接调用订阅者的回调事件，这是同步调用的。所以得出结论：观察者的监听方法调用是由数据发送的那个线程执行。

```java
// ObservableCreate.java 源码, onNext 是同步执行的
static final class CreateEmitter<T> extends AtomicReference<Disposable> implements ObservableEmitter<T>, Disposable {
    CreateEmitter(Observer<? super T> observer) {
        this.observer = observer;
    }

    // Emitter 中的 onNext 数据发送
    @Override
    public void onNext(T t) {
        if (t == null) {
            onError(ExceptionHelper.createNullPointerException("onNext called with a null value."));
            return;
        }
        if (!isDisposed()) {
            // 同步调用 Observer 的 onNext 监听方法
            observer.onNext(t);
        }
    }
}
```

### 1.2 Observer

Observer 观察者，通过实现三个方法来响应 Observable 数据流

+ onNext(data) 处理数据
+ onError(error) 处理异常，然后中断流
+ onComplete() 标记流的完成

## 2. Observable 的几个特点

每次订阅时，create 方法都会执行一遍，特殊情况下会有问题，如使用 http 访问远程数据时，每个观察者都会创建自己独立的 http 客户端，这会导致资源浪费。

Subject 可以解决这个问题，Subject 同时实现了 Observable 和 Observer 接口，即是观察者，又是一个可观察对象。Subject 会维护一个观察者列表。 通过引用 Subject，然后订阅真正的数据源，把自身当作一个中介，把上游的数据源发布到下游。

介绍几种 Subject

+ PublishSubject，实时的把上游收到的数据，转发到全部的订阅者。
  ![PublishSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/PublishSubject.png)

+ BehaviorSubject, 保留最近的一个数据给新观察者，后接收的数据实时转发到订阅者
  ![BehaviorSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.v3.png)

+ ReplaySubject, 重复所有历史数据给新观察者, 
  ![ReplaySubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.u.png)

+ AsyncSubject, 当上游数据完成时，取最后一个转发给订阅者
  ![AsyncSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/AsyncSubject.png)
