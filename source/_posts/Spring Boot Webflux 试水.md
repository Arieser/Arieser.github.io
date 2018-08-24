---
title: Spring Boot Webflux 试水
date: 2018-08-23 10:28:56
tags:
  - webflux
---

Spring 5 引入了反应式编程新特性，完全异步和非阻塞，简单了解以Spring 5为基础的 Spring Boot 2.0 构建的web应用，了解其异步的 、 非堵塞的 、 事件驱动服务的特性。下图是反应式编程和Spring MVC 架构的不同点。

![1535005541794](/images/1535005541794.png)

`WebFlux` 模块从上到下依次是 `Router Functions` 、 `WebFlux` 、 `Reactive Streams` 三个新组件

<!-- more -->

1. Router Functions

对标准的 `@Controller` ， `@RequestMapping` 等的 `Spring MVC` 注解，提供一套 **函数式风格** 的 `API` ，用于创建 `Router` 、 `Handler` 和 `Filter`.

2. WebFlux

   核心组件，协调上下游各个组件提供 **响应式编程** 支持

3. Reactive Streams

   一种支持 **背压** `(Backpressure)` 的 **异步数据流处理标准** ，主流实现有 `RxJava` 和 `Reactor` ， `Spring WebFlux` 集成的是 `Reactor`

**两种工作模式**

**Flux**

![flux](/images/qQvYBrB.png)

`Flux` 可以 触发 `(emit)` 很多 `item` ，而这些 `item` 可以经过若干 `Operators` 然后才被 `subscribe` ，下面是使用 `Flux` 的一个例子:

```java
Flux.fromIterable(getSomeLongList())
    .mergeWith(Flux.interval(100))
    .doOnNext(serviceA::someObserver)
    .map(d -> d * 2)
    .take(3)
    .onErrorResumeWith(errorHandler::fallback)
    .doAfterTerminate(serviceM::incrementTerminate)
    .subscribe(System.out::println);
```



**Mono**

![mono](/images/fAzY7nm.png)

`Mono` 只能触发 `(emit)` 一个 `item` ，下面是使用 `Mono` 的一个例子:

```java
Mono.fromCallable(System::currentTimeMillis)
    .flatMap(time -> Mono.first(serviceA.findRecent(time), serviceB.findRecent(time)))
    .timeout(Duration.ofSeconds(3), errorHandler::fallback)
    .doOnSuccess(r -> serviceM.incrementSuccess())
    .subscribe(System.out::println);
```

Mono 常用的方法有：

- Mono.create()：使用 MonoSink 来创建 Mono。
- Mono.justOrEmpty()：从一个 Optional 对象或 null 对象中创建 Mono。
- Mono.error()：创建一个只包含错误消息的 Mono。
- Mono.never()：创建一个不包含任何消息通知的 Mono。
- Mono.delay()：在指定的延迟时间之后，创建一个 Mono，产生数字 0 作为唯一值。



### Spring Boot 2.0 Reactive Stack

![ReactiveStack](/images/qAFFRbz.png)

从上到小两者的区别：

| API Stack  | Sevlet Stack             | Reactive Stack                    |
| ---------- | ------------------------ | --------------------------------- |
| Web控制层  | Spring MVC               | Spring WebFlux                    |
| 安全认证层 | Spring Security          | Spring Security                   |
| 数据访问层 | Spring Data Repositories | Spring Data Reactive Repositories |
| 容器API    | Servlet API              | Reactive Streams Adapters         |
| 内嵌容器   | Servlet Containers       | Netty, Servlet 3.1+ Containers    |



#### 适用场景

控制层一旦使用 `Spring WebFlux` ，它下面的安全认证层、数据访问层都必须使用 `Reactive API` 。其次， `Spring Data Reactive Repositories` 目前只支持 `MongoDB` 、 `Redis` 和 `Couchbase` 等几种不支持事务管理的 `NOSQL` 。技术选型时一定要权衡这些弊端和风险，比如：

1. `Spring MVC` 能满足场景的，就不需要更改为 `Spring WebFlux` 。
2. 要注意容器的支持，可以看看底层 **内嵌容器** 的支持。
3. 微服务体系结构， `Spring WebFlux` 和 `Spring MVC` 可以混合使用。尤其开发 `IO` **密集型** 服务的时候，可以选择 `Spring WebFlux` 去实现。

**注意点**：

1. 不能引入 `spring-boot-starter-web`依赖

reference

- [Spring Boot 2 初体验（1）--- 艰难的开始 ](https://github.com/xuexingdong/blog/issues/1)
- [实战Spring Boot 2.0 Reactive编程系列 - WebFlux初体验](https://www.imuo.com/a/a07f1cd4a4abd34bc308a0b4b5ff40507d83f1b63dc4118a9f8386667e971560)
- [Spring Boot 2.0 - WebFlux With MongoDB](https://my.oschina.net/u/3677020/blog/1579169)
- [Spring WebFlux快速上手——响应式Spring的道法术器](https://blog.csdn.net/get_set/article/details/79480233)

