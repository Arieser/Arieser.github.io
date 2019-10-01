---
title: Java8之扩展函数式接口
---

## 函数式接口

先看一下官方定义

> *Functional interfaces* provide target types for lambda expressions and method references.

可以看出函数式接口主要用于lambda表达式，这类接口只定义了唯一的抽象方法的接口（除了隐含的Object对象的公共方法），一开始也称*SAM*类型接口(Single Abstract Method)。

<!-- more -->

### 简单使用
```java
List<Integer> list = Lists.newArrayList(1,2,3);
        list.forEach(r -> {
            System.out.println("re = " + Math.sqrt(r));
        });
```

看一下  `foreach` 实现，在`Iterable.java`中

```java
default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

这里出现的`Consumer`就是一个函数式接口， java8 提供了一些常用的函数式接口

- Predicate -- 传入一个参数，返回一个bool结果， 方法为`boolean test(T t)`
- Consumer -- 传入一个参数，无返回值，纯消费。 方法为`void accept(T t)`
- Function -- 传入一个参数，返回一个结果，方法为`R apply(T t)`
- Supplier -- 无参数传入，返回一个结果，方法为`T get()`
- UnaryOperator -- 一元操作符， 继承Function,传入参数的类型和返回类型相同。
- BinaryOperator -- 二元操作符， 传入的两个参数的类型和返回类型相同， 继承BiFunction

这里就不一一列举了，具体请见 java.util.function 包 都很简单，不太清楚的自行google

### 扩展

但是jdk提供的有时候不一定能满足需求，这个时候就需要我们自定义函数式接口

1. 普通的 Function 或者 Consumer 只能就收一个参数，BiFuntion 和 BiConsumer 也只能接受连个参数，参数更多的情况就无法满足了

   以 consumer 为例，先自定义一个接口

   ```java
   @FunctionalInterface
       public interface TriConsumer<T, U, W> {
           void accept(T t, U u, W w);
   
           default TriConsumer<T, U, W> andThen(TriConsumer<? super T, ? super U, ? super W> after) {
               Objects.requireNonNull(after);
   
               return (l, r, w) -> {
                   accept(l, r, w);
                   after.accept(l, r, w);
               };
           }
       }
   ```

   函数式接口一般使用 @FunctionalInterface 注解注释，以申明该接口是一个函数式接口， 这里提供一个 andThen 方法以支持连续调用

   使用方法

   ```java
   TriConsumer<Integer, Integer, Integer> consumer = (a, b, c) -> {
               System.out.println(a + b + c);
           };
           consumer.accept(5,6,7);
   ```

   funtion类似，这里就不举例了

2. 异常捕获

   FunctionalInterface 提供的接口一般是不抛出异常的，意味着我们在使用的时候需要在方法体内部捕获异常，这里定义一种可以抛出异常的接口

   ```java
   @FunctionalInterface
       public interface InterfaceException<T> {
           void apply(T t) throws Exception;
       }
   ```





**References**

[Java 8函数式接口functional interface的秘密](<https://colobu.com/2014/10/28/secrets-of-java-8-functional-interface/>)