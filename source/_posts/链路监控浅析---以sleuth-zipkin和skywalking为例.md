--------------
title: 链路监控浅析---以sleuth-zipkin和skywalking为例
date: 2018-01-02 13:36:15
tags: 
	- JAVA
categories: 
--------------



**APM** (Application Performance Management & Monitoring)四种实现思路

1. 基于日志系统，探针只负责对日志加上编号，又类似ELK的系统进行收集、处理、展示。这方面没有很成熟的产品，一般都属于公司内部封装的框架。
2. 自动探针，适用语言：Java、C#、PHP、Node.js等等存在VM的语言。绝对大多数的商业产品和热门的开源产品都属于这个系列。
3. 全手动探针，优势是适用范围广，最有名的就是Zipkin的整个生态系统，分布式追踪几乎无处不在。也是现在全球运用最广泛的分布式监控系统。
4. 同时支持自动和手动模式的探针，适用语言同样是Java、C#、PHP、Node.js等等存在VM的语言，由于技术复杂性提高，运用的较少。优点是入门方便，同时使用灵活。商业上主要是Instana，开源主要是sky-walking提供了技术解决方案。

三大模块:  

1. **探针或sdk** ：负责数据采集和发送。探针或 SDK 是应用程序的收集端。一般使用插件的模式，自动探针一般是不需要修改程序，而 SDK 则是需要修改部分配置或者代码。skywalking 就是自动探针为主，zipkin-brave 就是 Zipkin 的 Java 手动探针
2. **collector模块** ：负责数据收集、分析、汇总、告警和存储。Collector 模块，这个根据不同的 APM 实现，可能由一个或者多个子系统构成。Collector 负责对探针和 SDK 提供网络接口（TCP、UDP、HTTP 不同形式接口）
3. **UI** ，负责高实时性展现。包括但不限于 Trace 的查询，统计数据展现，拓扑图展现，VM 或进程相关信息等，监控关键数据的展现

<!-- more -->

[监控概述-sleuth实现](http://www.cnblogs.com/softidea/p/7612570.html)

###  Sleuth-Zipkin

sleuth用来和zipkin(twitter)集成，自动完成span, trace等信息的生成，接入http request, 以及向zipkin server 发送采集信息，实现分布式服务跟踪能力。[全链路spring cloud sleuth+zipkin](http://blog.csdn.net/qq_15138455/article/details/72956232)

#### 结构图

![st1](/images/st1.png)

![st2](/images/st2.png)

![st3](/images/st3.png)

#### 搭建zipkin-server

1. 引入

   ```
   <dependency>
     <groupId>io.zipkin.java</groupId>
     <artifactId>zipkin-server</artifactId>
   </dependency>
   <dependency>
     <groupId>io.zipkin.java</groupId>
     <artifactId>zipkin-autoconfigure-ui</artifactId>
     <scope>runtime</scope>
   </dependency>
   ```

2. 添加注解 `@EnableZipkinServer`



#### rest服务调用

建立两个基本的rest服务，不再赘述

1. 引入

   ```
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-sleuth-zipkin</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-sleuth</artifactId>
   </dependency>
   ```


2. 添加配置

   `spring.zipkin.base-url=http://127.0.0.1:9418`



**启动**

LOG: aplication-name.TraceId.SpanId.BOOL



#### 其他配置

```properties
spring.sleuth.sampler.percentage=1
```



#### 数据持久化

![st4](/images/st4.png)

1. zipkin-server引入

   ```
   <!--此依赖会自动引入spring-cloud-sleuth-stream并且引入zipkin的依赖包(可以去除zipkin-server) -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-sleuth-zipkin-stream</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
   </dependency>
   <!--持久化数据到elasticsearch-->
   <dependency>
     <groupId>io.zipkin.java</groupId>
     <artifactId>zipkin-autoconfigure-storage-elasticsearch-http</artifactId>
     <version>2.4.2</version>
     <optional>true</optional>
   </dependency>
   ```


2. client引入

   `none`

3. 注解修改

   `@EnableZipkinStreamServer`

   ​

4. 问题

   4.1. 引入es后不能显示dependency tree [zipkin-dependencies](https://github.com/openzipkin/zipkin-dependencies)

   ```shell
   wget -O zipkin-dependencies.jar 'https://search.maven.org/remote_content?g=io.zipkin.dependencies&a=zipkin-dependencies&v=LATEST'
   STORAGE_TYPE=elasticsearch ES_HOSTS=host1,host2 java -jar zipkin-dependencies.jar
   ```


5. 概念

- **Span** ：基本工作单元，发送一个远程调度任务 就会产生一个Span，Span是一个64位ID唯一标识的，Trace是用另一个64位ID唯一标识的，Span还有其他数据信息，比如摘要、时间戳事件、Span的ID、以及进度ID
- **Trace** ：一系列Span组成的一个树状结构。请求一个微服务系统的API接口，这个API接口，需要调用多个微服务，调用每个微服务都会产生一个新的Span，所有由这个请求产生的Span组成了这个Trace。
- **Annotation** ：用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束 。这些注解包括以下： 
  - cs - Client Sent -客户端发送一个请求，这个注解描述了这个Span的开始
  - sr - Server Received -服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络传输的时间
  - ss - Server Sent （服务端发送响应）–该注解表明请求处理的完成(当请求返回客户端)，如果ss的时间戳减去sr时间戳，就可以得到服务器请求的时间。
  - cr - Client Received （客户端接收响应）-此时Span的结束，如果cr的时间戳减去cs时间戳便可以得到整个请求所消耗的时间。
- **Log**: span跨度内发生的事件，异常等，包含tag, annotation, log, event等






### sky-walking

提供分布式事务跟踪，以及APM性能监控。 [skywalking](https://github.com/apache/incubator-skywalking)， 使用[javaagent](http://www.kailing.pub/article/index/arcid/178.html)技术使得应用监控0耦合

#### 架构

![sw1](/images/sw1.png)



1. 部署
   - 下载agent, collector, ui 三个组件
   - 启动collector, ui
   - 修改application-name, 启动agent `-javaagent:/path/to/skywalking-agent/skywalking-agent.jar`






其他的一些APM工具: **Pinpoint**,  **CAT**, **Xhprof/Xhgui**

