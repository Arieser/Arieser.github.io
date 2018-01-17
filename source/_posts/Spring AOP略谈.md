---
title: Spring AOP略谈
date: 2016-12-16 22:36:43
tags: 
  - JAVA
---



AOP为Aspect Oriented Pragramming 的缩写，合理的使用AOP可以极大地提高程序的可重用性，同时提高开发的效率。

<!-- more -->

### 实现原理

Spring AOP实现主要是基于动态代理技术，实质上是Spring利用动态代理技术生成一个代理类型，重写了原目标组件方法的功能，在代理类中调用方面对象功能和目标对象功能。

### 常见的通知

- 前置通知：先执行AOP功能再执行目标功能
- 后置通知：目标无异常，在执行目标功能后执行方面功能
- 最终通知：目标有无异常均执行
- 异常通知：目标抛出异常后再执行方面功能
- 环绕通知：先执行方面前置部分，然后执行目标，最后执行方面后置部分

#### 前置通知

applicationContext.xml配置

```Java
<aop:config>
    <aop:aspect ref="optlogger">
        <aop:before method="log" point-cut="within(controller...)"/>
    </aop:aspect>
</aop:config>
```

**aop:after-returning, aop:after分别表示后置通知和最终通知**

#### 环绕通知

```Java
<aop:aspect ref="optlogger">
    <aop:around method="log" point-cut="within(controller...*)" />
</aop:aspect>
```

#### 异常通知
```Java
<aop:aspect ref="optlogger">
    <aop:after-throwing method="log" throwing="e" point-cut="within(controller...*)" />
</aop:aspect>
```

### 开发步骤

1. 开启AOP注解扫描

    - applicationContext.xml: `<aop:aspectj-autoproxy proxy-target-class="true" />`

    - 使用@Component注解标识，声明为组件

    - 使用@Aspect注解标识，声明为方面组件

2. 不同通知的注解方式

    - 前置通知：`@Before("within(controller...*)")`

    - 后置通知：`@AfterReturning("within(controller...*)")`

    - 最终通知：`@After("within(controller...*)")`

    - 环绕通知：`@Around("within(controller...*)")`

    - 异常通知：`@AfterThrowing(pointcut="within(controller...*)", throwing="e")`


