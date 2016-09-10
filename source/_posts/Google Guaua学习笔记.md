-----
title: Google Guaua学习笔记(一)
Date: 2016-09-10 11:47:04
tags: 
categories:
    - Java 
-----

### 集合[Collections]
Guaua对JDK集合的扩展，应用较广。
<!-- more -->
#### 不可变集合
Example:
```
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
        "red",
        "orange",
        "yellow",
        "green",
        "blue",
        "purple");

class Foo {
    Set<Bar> bars;
    Foo(Set<Bar> bars) {
        this.bars = ImmutableSet.copyOf(bars); // defensive copy!
    }
}
```

**所有Guava不可变集合的实现都不接受null值。我们对Google内部的代码库做过详细研究，发现只有5%的情况需要在集合中允许null元素，剩下的95%场景都是遇到null值就快速失败。如果你需要在不可变集合中使用null，请使用JDK中的Collections.unmodifiableXXX方法。更多细节建议请参考“使用和避免null”**

**构造方法**
- copyOf方法，如ImmutableSet.copyOf(set)
- of方法，如ImmutableSet.of(“a”, “b”, “c”)或 ImmutableMap.of(“a”, 1, “b”, 2);
- Builder工具，如`ImmutableSet.<Color>builder().addAll(WEBSAFE_COLORS).add(new Color(0, 191, 255)).build()`
- asList()


**细节**：关联可变集合和不可变集合

|可变集合接口 | 属于JDK还是Guava  |  不可变版本|
|:--------|:---------|:---------|
|Collection | JDK |ImmutableCollection|
|List |   JDK |ImmutableList|
|Set |JDK |ImmutableSet|
|SortedSet/NavigableSet  |JDK |ImmutableSortedSet|
|Map| JDK |ImmutableMap|
|SortedMap|   JDK| ImmutableSortedMap|
|Multiset|    Guava|   ImmutableMultiset|
|SortedMultiset|  Guava |  ImmutableSortedMultiset|
|Multimap |   Guava|   ImmutableMultimap|
|ListMultimap |   Guava|   ImmutableListMultimap|
|SetMultimap| Guava|   ImmutableSetMultimap|
|BiMap|   Guava|   ImmutableBiMap|
|ClassToInstanceMap|  Guava|   ImmutableClassToInstanceMap|
|Table|   Guava|   ImmutableTable|




**Reference**
[Guaua参考文档](http://ifeve.com/category/guava-2/)
