---
title: Java虚拟机(三)：性能调优分析
date: 2018-01-16 11:47:00
---

Java 应用性能的瓶颈点非常多，比如磁盘、内存、网络 I/O 等系统因素，Java 应用代码，JVM，GC，数据库，缓存等。笔者根据个人经验，将 Java 性能优化分为 4 个层级：应用层、数据库层、框架层、JVM 层。

<!--more-->

##### 高性能硬件上的程序部署策略

##### 集群间同步导致的内存溢出

##### 堆外内存导致的溢出错误

除了Java堆和永久代之外，其他占用内存较多的区域：

- **Direct Memory**：可以通过`-XX:MaxDirectMemorySize`调整大小，内存不足时抛出：`OutOfMemoryError`或者`OutOfMemoryError: Direct buffer memory`。
- **线程堆栈**：可通过`-Xss`调整大小，内存不足时抛出`StackOverflowError`（纵向无法分配，即无法分配新的栈帧）或者`OutOfMemeoryError：unable to create new native thread`(横向无法分配，即无法建立新的线程)
- **Socket缓存区**：每个Socket连接都Receive和Send两个缓存区，分别占大约37KB和25KB内存，连接多的话这块内存占用也比较客观。如果无法分配，则可能抛出`IOException: Too many open files`异常。
- **JNI代码**：如果代码中使用JNI调用本地库，那本地库使用的内存也不在堆中。
- **虚拟机和GC**：虚拟机、GC的代码执行也要消耗一定的内存。

##### 外部命令导致系统缓慢

例子：通过Java的`Runtime.getRuntime().exec()`调用系统脚本

##### 服务器JVM进程崩溃

##### 不恰当数据结构导致内存占用过大

##### 由Windows虚拟内存导致的长时间停顿

案例：http://hllvm.group.iteye.com/group/topic/28745

命令：

- `-XX:+HeapDumpOnOutOfMemoryError`发生OOM异常时，发回heapdump文件。





**reference**:

[Java 应用性能调优实践](https://www.ibm.com/developerworks/cn/java/j-lo-performance-tuning-practice/index.html)



