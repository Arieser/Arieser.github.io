---
title: java浮点型精度丢失浅析
date: 2019-11-23 16:11:23
---



java浮点型数值在运算中会出现精度损失的情况，在业务要求比较高比如交易等场景，一版使用BigDecimal来解决精度丢失的情况。最近一个同事在使用BigDecimal时仍然出现了精度损失，简略记录一下

### 测试用例

代码如下

```java
@Test
    public void fd() {
        double abc = 0.56D;
        System.out.println("abc: " + abc);
        System.out.println("new BigDecimal(abc): " + new BigDecimal(abc));
        System.out.println("BigDecimal.valueOf(abc): " + BigDecimal.valueOf(abc));
    }
```

输出

```shell
abc: 0.56
new BigDecimal(abc): 0.560000000000000053290705182007513940334320068359375
BigDecimal.valueOf(abc): 0.56
```

可以看到在使用BigDecimal构造器转化浮点型仍然会有损失，而使用`valueOf`方法则不会出现精度损失。

### 深入源码

BigDecimal构造器，核心代码(`BigDecimal(double val)`)如下

```java
public BigDecimal(double val, MathContext mc) {
  .....
    long valBits = Double.doubleToLongBits(val);
    int sign = ((valBits >> 63) == 0 ? 1 : -1);
    int exponent = (int) ((valBits >> 52) & 0x7ffL);
    long significand = (exponent == 0
                      ? (valBits & ((1L << 52) - 1)) << 1
                      : (valBits & ((1L << 52) - 1)) | (1L << 52));
  exponent -= 1075;
  ...
}
```

划重点， `Double.doubleToLongBits`返回根据IEEE754浮点“双精度格式”位布局，返回指定浮点值的表示

`BigDecimal.valueOf`核心代码

```java
public static BigDecimal valueOf(double val) {
        return new BigDecimal(Double.toString(val));
    }
public BigDecimal(char[] in, int offset, int len, MathContext mc) {
  ....
}
```

可以看到使用valueOf方法实际上是吧double转为String，再调用string构造器的。

那么为什么使用`Double.doubleToLongBits`会出现精度损失，而使用string构造器不会呢。主要原因是BigDecimal使用**十进制(BigInteger)+小数点(scale)位置**来表示小数，而不是直接使用二进制，如`101.001 = 101001 * 0.1^3`，运算时会分成两部分，BigInteger间的运算以及小数点位置的更新，这里不再展开。

### 原理浅析

`Double.doubleToLongBits`为什么会出现精度损失呢，主要原因是因为浮点型不能用精确的二进制来表述，就如十进制不能准确描述无穷小数一样。

浮点型转化为二进制的算法是乘以2直到没有了小数为止，举个栗子，0.8表示成二进制

>  0.8*2=1.6 取整数部分 1
>
>  0.6*2=1.2   取整数部分 1
>
>  0.2*2=0.4   取整数部分 0
>
>  0.4*2=0.8   取整数部分 0

可以看到上述的计算过程出现循环了，所以说浮点型转化为二进制有时是不可能精确的。

### 结论

如果想要把浮点型转化为BigDecimal，尽量选择使用`valueOf`方法，而不是使用构造器。



**References**

[JAVA程序中Float和Double精度丢失问题](http://blog.sina.com.cn/s/blog_827d041701017ctm.html)