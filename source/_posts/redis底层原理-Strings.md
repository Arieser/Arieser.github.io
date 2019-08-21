---
title: redis底层原理-Strings
date: 2019-08-21 17:26:43
tags:
  - redis
---



翻译自 [Under the Hood of Redis: Strings](http://redisplanet.com/redis/under-the-hood-of-redis-strings/)

你知道简单string `strings` 在redis里占用了56 bytes的内存吗？

我会试图告诉你为什么，了解redis运行原理时非常重要的。当你试图构建一个高负载的应用显的尤为重要，同时，你很快就会理解你的redis实例为什么会消费大量的内存？

<!-- more -->

这篇文章主要介绍以下几个主题：

- strings 在redis里如何存储
- strings 的内部结构是什么样的
- redis 使用的优化机制
- 依据不同场景，如何有效的使用strings或者以此为基础的结构

Strings是redis里最常用的数据结构。 HSET/ZSET/LIST 在内部结构上都会增加一定的开销。过去一年，我在  [stackoverflow](http://stackoverflow.com/) 上浏览了大量关于redis的答案， 让我意识到大量的开发者并不理解reids的内存结构以及redis为高速所付出的代价。这是该系列的第一篇文章，讲解redis内部构造。redis数据结构会占用多少内存的问题实际和编译器，CPU以及redis使用的内存分配器相关（redis默认使用jemalloc）。以下的计算依赖64位 centos 上的redis 3.0.5 版本。

对于不编写或者不熟悉C/C++的开发者而言，理解可能上不太容易。在此我会简化概念以让你能理解计算过程。在C/C++语言里，当你声明unsigned int (4  bytes) 变量，编译器会分配8 bytes内存（64位架构）。jemalloc 内存分配器会优化查找新的内存块的速度，并对齐分配的内存。jemalloc 的内存分配策略运行良好，然而接下来我认为我应该使用简化的概念来描述。你请求24 bytes，分配32。你请求61，分配64。我做了深度的简化，希望你理解的更清楚。

Salvatore Sanfilippo’s (aka antirez）通过一种SDS的结构来解释strings：

```shell
+--------+-------------------------------+-----------+
| Header | Binary safe C alike string... | Null term |
+--------+-------------------------------+-----------+
         |
         `-> Pointer returned to the user.
```

这是一种简单的`C`结构，header 部分包含string数据部分和末尾0的实际大小和内存占用空间的信息。我们感兴趣的事sds strings header结构的成本，resize策略和内存分配的代价。

2015年7月4号，pull request  [a long history with the optimization of sds strings](https://github.com/antirez/redis/pull/2509), 被引入Redis 3.2，使sds headers部分内存占用大幅度降低（从16%到200%不等）。移除了redis里关于redis string 最大512MB的限制。所有的这些可能性都归功于string长度变化时，header的动态变更。strings长度在256 bytes以下时，header仅占用3 bytes，65kb以下时占用5 bytes，512MB以下时占用9 bytes，uint64_t(64 bit unsigned integer)以下时占用17 bytes。而这种变化可以减少redis server farm 19.3%的内存（～42 GB）。然而，在Redis 3.0.x 中简化为 8 bytes 加 末端零占用的1 byte。让我们评估一下string `strings`的内存占用：

```shell
16 (header) + 7 (string length) + 1(trailing zero) = 24  bytes (16  bytes in the header, because the compiler will align 2 unsigned int for you).
```

jemalloc 会分配32 bytes。Let’s take as long as it will not be taken into account （我希望你售后能理解为什么）。

当一个字符串大小变化时会引起什么变化？当你增加字符串长度，同时发现已分配的内存不足，redis 会将新长度和常量`SDS_MAX_PREALLOC`（sds.h中定义，值为1,048,576 bytes）比较。如果新长度比该值小，则会分配两倍的请求大小。如过请求长度大于 `SDS_MAX_PREALLOC` ，新增加的长度会增加到这个常量上。

这个特性对于主题-bitmaps使用中内存减少 这个问题非常重要。分配的内存通常是需要的两倍，是因为setbit实现的需要（参见 setbit 命令，bitops.c）。

现在你可以说 strings 会占用32 bytes（包括已分配的）。浏览过 hashedin.com ([redis memory optimization guide](https://github.com/sripathikrishnan/redis-rdb-tools/wiki/Redis-Memory-Optimization)) 的读者可能会想起他们被强烈建议不要使用少于100 bytes 的字符串，比如 `set foo bar` 会占用 ～96 bytes，其中 90 bytes 的开销（64位机器）。讲道理，让我们看一下为什么。

reids里所有的值都被命名为 `redisObject`， 内部结构如下：

```sh
+------+----------+-----+----------+-------------------------+
| Type | Encoding | LRU | RefCount | Pointer to  data (ptr*) |
+------+----------+-----+----------+-------------------------+
```

稍后我们会计算字符串的大小，了解账户编译器和jemalloc特性。了解存储字符串的编码是非常重要的，redis会使用三种不同的存储策略：

- **REDIS_ENCODING_INT**. Strings can be stored in this form, if the value is cast to long value in the range **LONG_MIN**, **LONG_MAX**. For example, the string «dict» it will be stored in the form of this encoding, and will be the number 1952672100 (0x74636964). This encoding is also used for pre-selected range of special values in the range **REDIS_SHARED_INTEGERS** (defined in redis.h and the default is 10000). The values of this range are allocated immediately at the start of Redis.
- **REDIS_ENCODING_EMBSTR** used for strings with a length up to 39 bytes (the value from constant **REDIS_ENCODING_EMBSTR_SIZE_LIMIT** object.c). This means that redisObject structure and sds string structure are placed in a single area of memory allocated by allocator. With this in mind, we will be able to calculate the correct alignment. However, it is equally important to understand the problem of memory fragmentation in the Redis and how to live with it.
- **REDIS_ENCODING_RAW** used for all strings whose length exceeds **REDIS_ENCODING_EMBSTR_SIZE_LIMIT**. In this case our ptr * stores a pointer to the memory area with sds string.

EMBSTR 在2012年出现，在短字符串方面，带来了大约 60%-70%的性能提升，但目前对内存及其碎片化影响的研究还不多。

7 bytes 的 `strings` 字符串，使用 EMBSTR 存储结构。构建的存储结构类似这样：

```shell
+--------------+--------------+------------+--------+----+
| robj data... | robj->ptr    | sds header | string | \0 |
+--------------+-----+--------+------------+--------+----+
                     |                       ^
                     +-----------------------+
```



现在我们可以再次计算 `strings` 的内存占用情况

```shell
(4 + 4)* + 8(encoding) + 8 (lru) + 8 (refcount) + 8 (ptr) + 16 (sds header) + 7(strig itself) + 1 (terminating zero) = 56 bytes.
```

*The type and value in redisObject uses only the 4 lower and higher bits in the same number, so these two aligned fields will take 8 bytes.*

让我们检查一下，使用 DEBUG SDSLEN 来debug SDS (http://redis.io/commands/debug-object) 字符串。这个命令在redis2.6 被加入。

```shell
set key strings
+OK
debug object key
+Value at:0x7fa037c35dc0 refcount:1 encoding:embstr serializedlength:8 lru:3802212 lru_seconds_idle:14
debug sdslen key
+key_sds_len:3, key_sds_avail:0, val_sds_len:7, val_sds_avail:0
```

使用EMBSTR编码，字符串长度 7 bytes（有效SDS长度），那么 hashdin.com 的开发者讨论的 96 bytes 又是关于什么呢？在我的理解中，他们犯了一点小错误，`set foo bar 需要分配112 bytes内存（value 56 bytes，key 56 bytes）`，内存开销 106 bytes。

我承诺会说明使用BITMAP时，节省内存的情况。Redis 2.2 开始出现的 Bit 和 byte 操作 就想一个实时计数的魔法棒，可以节省内存。官方口号是“上亿用户数据，仅占用12M内存“。

理解了redis内存字符串原理，也可以了解bitmap。“是否应该被用于少量数据？”。假设你需要记录一千万人的上网数据：

```shell
setbit online 10000000 1
:0
debug sdslen online
+key_sds_len:6, key_sds_avail:0, val_sds_len:1250001, val_sds_avail:1048576
```

你会消费 2,288,577 bytes 内存，对你来说“有用”的部分为 1,250,001 bytes。存储你的一个用户花费 ～2.3 MB，使用 SET 你需要 ～64 bytes（pyaload 为 4 bytes）。使用这种数据结构可以有效减少内存使用量。如果你有10,000～100,000用户，bitmap结构就可以复用内存。

最后，了解一下 字符串 resize，即就是重新分配内存块。内存碎片化是redis的另一个特性，很少有开发者能考虑到这一点：

```shell
info memory
$222
# Memory
used_memory:506920
used_memory_human:495.04K
used_memory_rss:7565312
used_memory_peak:2810024
used_memory_peak_human:2.68M
used_memory_lua:36864
mem_fragmentation_ratio:14.92
mem_allocator:jemalloc-3.6.0
```

`mem_fragmentation_ratio` 指标显示了系统分配的内存`used_memory_rss` 和 redis使用内存`used_memory`的比率。`use_memory` 和 `use_memory_ree`包含了数据和redis存储的内部数据结构所占用内存。 Redis RSS (`Resident Set Size`) - RAM allocated by the operating system, which in addition to the user data (and the costs of their internal representation) accounted for the cost of fragmentation during the physical allocation of the operating system.

如何理解 `mem_fragmentation_ratio` ？2.1 意思是需要210%的更多内存。小于1则意味着内存被终止，操作系统正在交换内存。

实际中，如果该数字超过 1-1.5边界意味着有地方出错了，尝试以下解决方法：

- 重启redis。redis越长时间不重启，这个值就会越大。
- 检查一下你计划存储的数据量。比方说，如果你使用32位redis存储多达4GB的数据，那么你应该使用64位redis以扩增rdb。
- 如果你了解内存分配器的不同点，可以考虑更换内存分配器。

其他资料：

- http://redis.io/topics/memory-optimization
- http://redis.io/topics/internals-sds
- http://redislabs.com/blog/redis-ram-ramifications
- http://github.com/sripathikrishnan/redis-rdb-tools/wiki/Redis-Memory-Optimization