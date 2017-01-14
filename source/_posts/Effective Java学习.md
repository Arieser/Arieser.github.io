-----
title: Effective Java学习
Date: 2016-08-25 17:36:30
tags: 
categories: 
    - Java
-----

为进一步提高java基础水平，开始研读effective java，简略记录一下阅读过程中的疑问和心得。

<!-- more -->

#### 考虑用静态工厂方法代替构造器


#### 构造器模式：重叠构造器模式， JavaBean模式，buider方法

1. 重叠构造器模式即构建多个构造器，但是当有多个参数是，代码会很难编码，阅读困难。
2. JavaBean模式：调用一个无参构造器来创建对象，然后调用setter方法来设置每个必要的参数：构造过程中JavaBean可能处于不一致的状态；同时阻止了把类做成不可变的可能，可能影响线程安全。
3. Builder模式：不直接生成对象，通过一个间接的builder对象调用类似于setter方法来设置可选参数。易编写同时也易于阅读。适用于多个参数

``` Java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        private final int servingSize;
        private final int servings;

        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        carbohydrate = builder.carbohydrate;
        fat = builder.fat;
        sodium = builder.sodium;
    }
}
```

实现示例：httpclient的RequestConfig

#### 避免创建不必要的对象

一个极端的例子：
`String s = new String("Stringabcd")            // Don't do this!`

