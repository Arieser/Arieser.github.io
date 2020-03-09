---
title: 浅析Java8 CompletableFuture和Stream串行使用产生的一个问题
date: 2019-11-2 13:36:15
tags: 

  - java
---

#### 问题来源

问题来源于一次串行使用CompletableFuture和Stream导致CompletableFuture异步失效的问题，问题代码：

```java
int size = 50;
        List<Double> res = Stream.iterate(0, i -> i + 1).limit(size).map(i -> CompletableFuture.supplyAsync(() -> {
            Double re = BigDecimal.valueOf(10 * (Math.sin(i) + 1)).setScale(2, RoundingMode.HALF_UP).stripTrailingZeros().doubleValue();
            try {
                TimeUnit.SECONDS.sleep(re.intValue());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return re;
        }, ThreadUtil.fixed()))
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
```

其中`ThreadUtil.fixed()`是自己封装的线程工具方法，也可以使用`Executors.newFixedThreadPool`代替，原本是想通过CompletableFuture拿到异步执行的结果并进行处理，串联使用stream后反而未起到作用，先说解决方法：

1. 先收集 future 结果到list，再调用新的流运算，即 `.map(CompletableFuture::join)`方法
2. `limit(size).map` 之间添加 parallel() 方法，形成 `parallelStream()`d的形式

原因分析：普通的stram可以理解为单纯的foreach循环，每生成一个future立即join，出现异步变同步的现象。

再进一步，既然CompletableFuture和parellStream都可以并行执行任务，有必要比较一下。

#### parallelStream

先试用 parallelStream 重写上述方法

```java
List<Double> result = Stream.iterate(0, i -> i + 1).limit(50).parallel().map(i -> {
            ///.....
        }).collect(Collectors.toList());
```

用时：54018ms

想要深入了解parallelStream，需要先了解ForkJoin框架和ForkJoinPool框架。这里简单介绍一下ForkJoinPool，真正了解 ParallelStream 还是需要先弄懂ForkJoinPool的，在此只是简单比较两者功能，不做深入探讨。

ForkJoinPool 使用分治法(Divide-and-Conquer Algorithm)来解决问题，实现了ExecutorService接口，线程数量可以通过构造器传入，默认使用机器的CPU数量。和ThreadPoolExecutor有一定区别，ForkJoinPool可以在运行线程中创建新的任务，并挂起当前的任务，此时线程就能够从队列中选择子任务执行，而ThreadPoolExecutor做不到这一点的。ForkJoinPool的核心算法是工作窃取算法，这样就可以在使用少量的线程来完成大量的任务。比如说ForkJoinPool 4个线程可以处理200完个任务，ThreadPoolExecutor显然是不可行的。

> 工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。

ParallelStreams中java8为ForkJoinPool添加了通用线程池，默认线程数量为机器的处理器数量。可以通过 `-Djava.util.concurrent.ForkJoinPool.common.parallelism=N`来设置ForkJoinPool的线程数量。

#### 拆分成两步重写上述方法

```java
List<CompletableFuture<Double>> futures = Stream.iterate(0, i -> i + 1).limit(50).map(i -> CompletableFuture.supplyAsync(() -> {
            ///.....
        }, ThreadUtil.fixed())).collect(Collectors.toList());
        List<Double> ids = futures.stream().map(CompletableFuture::join)
                .collect(Collectors.toList());
```

用时：57111ms， 通过增加线程数量可以减少执行时间。



References

[Java 多线程中的任务分解机制-ForkJoinPool，以及CompletableFuture](https://www.cnblogs.com/hongdada/p/8876028.html)

[CompletableFuture 组合式异步编程](https://blog.csdn.net/itguangit/article/details/78624404)



