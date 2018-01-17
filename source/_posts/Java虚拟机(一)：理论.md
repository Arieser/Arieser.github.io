---
title: Java虚拟机(一)：理论
date: 2018-01-11 19:40:00
---

简要记录一些重要概念和常见问题，后续逐步补充。

<!--more-->

#### 运行时数据区域

![vm-runtime-data](/images/vm-runtime-data.png)

**栈**：分为虚拟机栈和本地方法栈，存放局部变量表部分。

**堆**：存放对象实例，GC管理的主要区域。

**程序计数器**：是一块较小的内存空间，可以看作是所执行的字节码的行号指示器。通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

**方法区**(非堆)：各个线程共享的内存区域，用于存储被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。



####  OOM异常

##### 堆溢出

```java
/**
 * VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError 
 */
public class HeapOOM {
    
    static class OOMObject{
    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

出现异常`java.lang.OutOfMemoryError: Java heap space`



##### 栈溢出

```java
/**
 * VM Args: -Xss128k
 */
public class JavaVMStackSOF {
    private Integer stackLength = 1;
    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) {
        JavaVMStackSOF oom = new JavaVMStackSOF();
        try {
            oom.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack Length: " + oom.stackLength);
            throw e;
        }
    }
}
```

出现 `java.lang.StackOverflowError`



#### 问题

1. 判定对象是否存活？
   - **引用计数算法**：给对象添加一个引用计数器，每当一个地方引用它时，计数器加1；当引用失效时，计数器值减1；任何时刻计数器为0的对象就是不可能再被使用的。缺点：很难解决对象之间相互循环引用的问题。
   - **可达性分析**：基本思路是通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连（用图论的话来说，就是从GC Roots到这个对象不可达）时，则证明此对象是不可用的。
     - 可作为GC Roots的对象：
       - 虚拟机栈（栈帧中的本地变量表）中引用的对象。
       - 方法区中类静态属性引用的对象。
       - 方法区中常量引用的对象。
       - 本地方法栈中JNI（即一般说的Native方法）引用的对象。

2. 引用的种类
   - 强引用（Strong Reference）
   - 软引用（Soft Reference）
   - 弱引用（Weak Reference）
   - 虚引用（Phantom Reference）

3. 内存分配规则


   - 对象优先在Eden分配：使用 `-XX:+PrintGCDetails`打印虚拟机发生垃圾收集行为时打印内存回收日志，并且在进程退出的时候输出当前的内存各区域分配情况。空间不足，则发起一次Minor GC。

     ```java
     // vm args: -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+PrintGCDetails
     public static void testAllocation() {
       byte[] a1, a2, a3, a4;
       a1 = new byte[2 * _1MB];
       a2 = new byte[2 * _1MB];
       a3 = new byte[2 * _1MB];
       a4 = new byte[4 * _1MB]; // 出现一次MinorGC
     }
     ```

   - 大对象直接进入老年代：使用`-XX:PretenureSizeThreshold=X`令大于这个设置值的对象直接在老年代分配

     ```java
     // vm args: -verbose:gc -Xms20M -Xmx20M -Xmn20M -XX:SurvivorRatio=8 -XX:+PrintGCDetails
     // -XX:PretenureSizeThreshold=3145728
     public static void testPretenureSizeThreshold() {
       byte[] allocation;
       allocation = new byte[6 * _1MB];
     }
     ```

   - 长期存活的对象将进入老年代：如果对象在Eden出生并经过第一次Minor GC 后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。对象在Survivor区中每“熬过”一次Minor GC，年龄就增加1岁，当他的年龄增加到一定程度（默认为15岁），就将会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数： `-XX:MaxTenuringThreshold`设置。

     ```java
     // vm args: -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+PrintGCDetails -XX:MaxTenuringThreshold=1
     // -XX:+PrintTenuringDistribution
     public static void testTenuringThreshold() {
       byte[] allocation1, allocation2, allocation3;
       allocation1 = new byte[_1MB / 4];
       allocation2 = new byte[4 * _1MB];
       allocation3 = new byte[4 * _1MB];
       allocation3 = null;
       allocation3 = new byte[4 * _1MB];
     }
     ```

   - 动态对象年龄判定：如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。

   - 空间分配担保：虚拟机在进行Minor GC之前检查老年代最大可用的连续空间是否大于新生代所有对象总空间，以确保Minor GC是安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均 大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是由风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时候也要改为进行一次FullGC。