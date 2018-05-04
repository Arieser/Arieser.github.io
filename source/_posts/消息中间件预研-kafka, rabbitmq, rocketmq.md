---
title: 消息中间件预研-rabbitmq, rocketmq
date: 2018-01-04 10:36:15
tags:
  - JAVA
---



消息中间件在服务开发中起着重要的作用，应对业务需求，对rabbitmq，rocketmq进行预研，kafka暂时不做深入了解。

<!--more-->

**消息中间件应用场景**

1. 可以做延迟设计
   比如我们有一些数据，需要过五分钟后再被使用，这时候就需要使用延迟队列设计，比如在RabbitMQ中利用死信队列实现。
   具体实现在这里：<http://www.cnblogs.com/haoxinyue/p/6613706.html>

2. 异步处理
   这个场景主要应用在多任务执行的场景。

3. 应用解耦
   在大型微服务架构中，有一些无状态的服务经常考虑使用mq做消息通知和转换。

4. 分布式事务最终一致性
   可以使用基于消息中间件的队列做分布式事务的消息补偿，实现最终一致性。

5. 流量削峰
   一般在秒杀或团抢活动中使用广泛，可以通过队列实现秒杀的人数和商品控制，还可以缓解短时间压垮应用系统。

6. 日志处理
   我们在做监控，或者日志采集的时候经常用队列来做消息的传输和暂存。




<!--more-->


### RocketMQ(Apache 4.2.0)

#### 概念

Disk Flush (磁盘刷新/同步操作):  就是将内存的数据落地，存储在磁盘中.

- **SYNC_FLUSH（同步刷盘）：**生产者发送的每一条消息都在保存到磁盘成功后才返回告诉生产者成功。这种方式不会存在消息丢失的问题，但是有很大的磁盘IO开销，性能有一定影响。
- **ASYNC_FLUSH（异步刷盘）：**生产者发送的每一条消息并不是立即保存到磁盘，而是暂时缓存起来，然后就返回生产者成功。随后再异步的将缓存数据保存到磁盘，有两种情况：1是定期将缓存中更新的数据进行刷盘，2是当缓存中更新的数据条数达到某一设定值后进行刷盘。这种方式会存在消息丢失（在还未来得及同步到磁盘的时候宕机），但是性能很好。默认是这种模式。

Broker Replication (Broker间数据同步/复制): 集群环境下需要部署多个Broker，Broker分为两种角色：一种是master，即可以写也可以读，其brokerId=0，只能有一个；另外一种是slave，只允许读，其brokerId为非0。一个master与多个slave通过指定相同的brokerName被归为一个broker set（broker集）。通常生产环境中，我们至少需要2个broker set。

Broker Replication只的就是slave获取或者是复制master的数据.

- **Sync Broker：**生产者发送的每一条消息都至少同步复制到一个slave后才返回告诉生产者成功，即“同步双写”。
- **Async Broker：**生产者发送的每一条消息只要写入master就返回告诉生产者成功。然后再“异步复制”到slave。

#### start [十分钟入门RocketMQ](http://jm.taobao.org/2017/01/12/rocketmq-quick-start-in-10-minutes/)

**QuickStart**: [apache-quickstart](https://rocketmq.apache.org/docs/quick-start/)



![rocketmq](/images/rocketmq.png)



#### 集群部署

- 使用不同配置文件启动nameserv(默认9876)

  无状态节点，可集群部署，**节点之间无任何信息同步**（Broker与每个namesrv连接，可以保证信息同步性）

  nameserv的所有配置信息

  ```properties
  rocketmqHome=/usr/local/rocketmq
  kvConfigPath=/Users/zhangyanghong/namesrv/kvConfig.json
  productEnvName=center
  clusterTest=false
  orderMessageEnable=false
  listenPort=9876
  serverWorkerThreads=8
  serverCallbackExecutorThreads=0
  serverSelectorThreads=3
  serverOnewaySemaphoreValue=256
  serverAsyncSemaphoreValue=64
  serverChannelMaxIdleTimeSeconds=120
  serverSocketSndBufSize=4096
  serverSocketRcvBufSize=4096
  serverPooledByteBufAllocatorEnable=true
  useEpollNativeSelector=false
  ```

  通过修改listenPort在一台机器上部署启动两个nameserv

  `nohup sh mqnamesrv -c mqnamesrv-a.conf &`

- 启动broker(默认10911)集群

  broker的所用配置项

  ```properties
  rocketmqHome=/usr/local/rocketmq
  namesrvAddr=
  ## 本机ip地址，默认系统自动识别
  brokerIP1=172.18.48.79
  brokerIP2=172.18.48.79
  brokerName=jiexiu’Mac
  brokerClusterName=DefaultCluster
  brokerId=0
  brokerPermission=6
  defaultTopicQueueNums=8
  autoCreateTopicEnable=true
  clusterTopicEnable=true
  brokerTopicEnable=true
  autoCreateSubscriptionGroup=true
  messageStorePlugIn=
  sendMessageThreadPoolNums=32
  pullMessageThreadPoolNums=24
  adminBrokerThreadPoolNums=16
  clientManageThreadPoolNums=16
  flushConsumerOffsetInterval=5000
  flushConsumerOffsetHistoryInterval=60000
  rejectTransactionMessage=false
  fetchNamesrvAddrByAddressServer=false
  sendThreadPoolQueueCapacity=10000
  pullThreadPoolQueueCapacity=10000
  filterServerNums=0
  longPollingEnable=true
  shortPollingTimeMills=1000
  notifyConsumerIdsChangedEnable=true
  highSpeedMode=false
  commercialEnable=true
  commercialTimerCount=1
  commercialTransCount=1
  commercialBigCount=1
  transferMsgByHeap=true
  maxDelayTime=40
  regionId=DefaultRegion
  registerBrokerTimeoutMills=6000
  slaveReadEnable=false
  disableConsumeIfConsumerReadSlowly=false
  consumerFallbehindThreshold=0
  waitTimeMillsInSendQueue=200
  startAcceptSendRequestTimeStamp=0
  listenPort=10911
  serverWorkerThreads=8
  serverCallbackExecutorThreads=0
  serverSelectorThreads=3
  serverOnewaySemaphoreValue=256
  serverAsyncSemaphoreValue=64
  serverChannelMaxIdleTimeSeconds=120
  serverSocketSndBufSize=131072
  serverSocketRcvBufSize=131072
  serverPooledByteBufAllocatorEnable=true
  useEpollNativeSelector=false
  clientWorkerThreads=4
  clientCallbackExecutorThreads=4
  clientOnewaySemaphoreValue=65535
  clientAsyncSemaphoreValue=65535
  connectTimeoutMillis=3000
  channelNotActiveInterval=60000
  clientChannelMaxIdleTimeSeconds=120
  clientSocketSndBufSize=131072
  clientSocketRcvBufSize=131072
  clientPooledByteBufAllocatorEnable=false
  clientCloseSocketIfTimeout=false
  storePathRootDir=/Users/zhangyanghong/store
  storePathCommitLog=/Users/zhangyanghong/store/commitlog
  mapedFileSizeCommitLog=1073741824
  mapedFileSizeConsumeQueue=6000000
  flushIntervalCommitLog=1000
  flushCommitLogTimed=false
  flushIntervalConsumeQueue=1000
  cleanResourceInterval=10000
  deleteCommitLogFilesInterval=100
  deleteConsumeQueueFilesInterval=100
  destroyMapedFileIntervalForcibly=120000
  redeleteHangedFileInterval=120000
  ## 删除时间点，默认凌晨4点
  deleteWhen=04
  diskMaxUsedSpaceRatio=75
  ## 文件保留时间，默认48小时
  fileReservedTime=72
  putMsgIndexHightWater=600000
  maxMessageSize=4194304
  checkCRCOnRecover=true
  flushCommitLogLeastPages=4
  flushLeastPagesWhenWarmMapedFile=4096
  flushConsumeQueueLeastPages=2
  flushCommitLogThoroughInterval=10000
  flushConsumeQueueThoroughInterval=60000
  maxTransferBytesOnMessageInMemory=262144
  maxTransferCountOnMessageInMemory=32
  maxTransferBytesOnMessageInDisk=65536
  maxTransferCountOnMessageInDisk=8
  accessMessageInMemoryMaxRatio=40
  ## 是否开启消息索引功能
  messageIndexEnable=true
  maxHashSlotNum=5000000
  maxIndexNum=20000000
  maxMsgsNumBatch=64
  ## 是否提供安全的消息索引机制，索引保证不丢
  messageIndexSafe=false
  haListenPort=10912
  haSendHeartbeatInterval=5000
  haHousekeepingInterval=20000
  haTransferBatchSize=32768
  haMasterAddress=
  haSlaveFallbehindMax=268435456
  ## Broker的角色：ASYNC_MASTER异步复制Master; SYNC_MASTER同步双写MASTER; SLAVE
  brokerRole=ASYNC_MASTER
  flushDiskType=ASYNC_FLUSH
  syncFlushTimeout=5000
  messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
  flushDelayOffsetInterval=10000
  cleanFileForciblyEnable=true
  warmMapedFileEnable=false
  offsetCheckInSlave=false
  debugLockEnable=false
  duplicationEnable=false
  diskFallRecorded=true
  osPageCacheBusyTimeOutMills=1000
  defaultQueryMaxNum=32
  ```

  其中重要的配置信息如下：
  ```properties
  namesrvAddr=
  brokerIP1=172.18.48.79
  brokerName=jiexiu’Mac
  brokerClusterName=DefaultCluster
  brokerId=0
  autoCreateTopicEnable=true
  autoCreateSubscriptionGroup=true
  rejectTransactionMessage=false
  fetchNamesrvAddrByAddressServer=false
  storePathRootDir=/Users/zhangyanghong/store
  storePathCommitLog=/Users/zhangyanghong/store/commitlog
  flushIntervalCommitLog=1000
  flushCommitLogTimed=false
  deleteWhen=04
  fileReservedTime=72
  maxTransferBytesOnMessageInMemory=262144
  maxTransferCountOnMessageInMemory=32
  maxTransferBytesOnMessageInDisk=65536
  maxTransferCountOnMessageInDisk=8
  accessMessageInMemoryMaxRatio=40
  messageIndexEnable=true
  messageIndexSafe=false
  haMasterAddress=
  brokerRole=ASYNC_MASTER
  flushDiskType=ASYNC_FLUSH
  cleanFileForciblyEnable=true
  ```

  - 启动broker

    `sh mqbroker -n '127.0.0.1:9876;127.0.0.1:9877' -c ../conf/2m-noslave/broker-a.properties  > /dev/null 2>&1 &`

- 集群验证

  `sh mqadmin clusterList -n 127.0.0.1:9876 `

  输出信息

  ```shell
  #Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE
  DefaultCluster    broker-a                0     172.18.48.79:10911     V3_5_8                   0.00(0,0ms)         0.00(0,0ms)          0 412299.47 0.5476
  DefaultCluster    broker-b                0     172.18.48.79:12911     V3_5_8                   0.00(0,0ms)         0.00(0,0ms)          0 412299.47 0.5476
  ```

  ​

- 默认的集群配置conf子目录下

  ```shell
  2m-2s-async     // 两个master 两个slave异步
  2m-2s-sync		// 两个master 两个slave同步
  2m-noslave		// 两个master 没有slave
  ```

- [broker 的master和slave (Slave 不可写，但可读)](http://blog.csdn.net/zhu_tianwei/article/details/40949523)

  - **单个master**: 风险较大, 不建议生产使用
  - **多master**: 
    - 配置简单，消息可靠，性能最高
    - 单台机器宕机期间，未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响
  - **多master多slave，异步复制**:
    - 每个 Master 配置一个 Slave，有多对Master-Slave，HA 采用异步复制方式，主备有短暂消息延迟，毫秒级
    - 优点:  即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，因为 Master 宕机后，消费者仍然可以从 Slave 消费，此过程对应用透明。不需要人工干预。性能同多 Master 模式几乎一样
    - 缺点: Master 宕机，磁盘损坏情况，会丢失少量消息
  - **多master多slave，同步双写**:
    - 每个 Master 配置一个 Slave，有多对Master-Slave，HA 采用同步双写方式，主备都写成功，向应用返回成功
    - 优点: 数据与服务都无单点，Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高
    - 缺点: 性能比异步复制模式略低，大约低 10%左右，发送单个消息的 RT 会略高。目前主宕机后，备机不能自动切换为主机，后续会支持自动切换功能。

- 问题

  1. 出现 `Lock failed,MQ already started`

     **解决方案**: 修改配置文件中的`storePathRootDir`项

  2. 启动 mqnamesrv / mqbroker服务报错，显示内存不够，需大于2G？具体表现: “*VM warning: INFO: OS::commit_memory(0x00000006c0000000, 2147483648, 0) faild; error=’Cannot allocate memory’ (errno=12)*”

     **解决方案**：修改`/RocketMQ/devnev/bin/` 下的服务启动脚本 `runserver.sh` 、`runbroker.sh` 中对于内存的限制，改成如下示例：

     `JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=128m"`

     ​

#### 启动rocketmq-console

`mvn spring-boot:run`

or

`java -jar rocketmq-console-ng-1.0.0.jar --server.port=12581 --rocketmq.config.namesrvAddr=10.89.0.64:9876;10.89.0.65:9876  `



#### topic

rocketmq中一个broker-name其实就相当于kafka-broker中的一个partition，而rocketmq每一个slave就相当于kafka中的一个replication，这种情况，所以rocketmq的特点相当于单个partition支持多队列，大致的原理图如下：

![rocketmq-topic](/images/rocketmq-topic.png)

默认: 一个topic的队列数是8



#### 问题 

1. producer生产消息过多，customer来不及消费?

2. 消息的重试机制

3. broker和client/producer版本不一致问题

   解决: 会导致已消费消息堆积，重启customer会重复消费，更换一致版本

4. 队列个数设置

   producer发送消息时候设置，特别注意：同一个topic仅当第一次创建的时候设置有效，以后修改无效，除非修改broker服务器上的consume.json文件，

   demo：`mqProducer.setDefaultTopicQueueNums(5)`

   参考：http://www.mamicode.com/info-detail-327693.html

5. [其他常见问题3.2.4](http://www.iteye.com/topic/1146280)





### RabbitMQ



#### 安装erlang

- 使用kerl安装和管理erlang，参考 [Erlang版本管理工具: Kerl](https://segmentfault.com/a/1190000004909357)  ,  [安装Erlang/OTP的简单方法](https://www.jianshu.com/p/caddaa8251af), 其他安装方法 [在CentOS上安装erlang](https://zfanw.com/blog/install-erlang-on-centos-7.html)
- 设置环境变量



#### 安装 RabbitMQ

- 下载rabbit rpm包

- 错误

  ```shell
  Error: Package: rabbitmq-server-3.7.2-1.el7.noarch (/rabbitmq-server-3.7.2-1.el7.noarch)
             Requires: erlang >= 19.3
   You could try using --skip-broken to work around the problem
   You could try running: rpm -Va --nofiles --nodigest
  ```

  执行 `rpm --nodeps -ivh rabbitmq-server-3.7.2-1.el7.noarch.rpm`

- 启动命令(/usr/lib/rabbitmq /etc/rabbitmq  ---- /usr/share/doc/rabbitmq-server-3.7.2)

  ```shell
  service rabbitmq-server start
  service rabbitmq-server stop
  service rabbitmq-server restart
  service rabbitmq-server status
  ./rabbitmqctl stop 
  ```

- 开启管理功能

  `rabbitmq-plugins enable rabbitmq_management`

- 启动服务

  `rabbitmq-server -detached`

- 添加用户权限

  ```shell
  rabbitmqctl add_user albert password
  rabbitmqctl set_user_tags albert administrator
  ```

- 开机自启动

  `chkconfig rabbitmq-server on`

- 修改配置文件，开启远程用户访问

  ```shell
  cp /usr/share/doc/rabbitmq-server-3.7.2/rabbitmq.config.example /etc/rabbitmq/  
  mv rabbitmq.config.example rabbitmq.config 
  ```

  增加 

  `{loopback_users, []}`

- 集群部署

  [RabbitMQ 配置初步](https://blog.apporc.org/2016/04/rabbitmq-%E9%85%8D%E7%BD%AE%E5%88%9D%E6%AD%A5/)

  [Centos7 安装rabbitmq](http://blog.csdn.net/wenyu826/article/details/71108279)

- reference

  [How-to-install-rabbitmq-on-centos-7](https://www.vultr.com/docs/how-to-install-rabbitmq-on-centos-7)

  ​



#### 重要概念: 

![rabbit](/images/rabbit.png)

- 左侧 P 代表 生产者，也就是往 RabbitMQ 发消息的程序。
- 中间即是 RabbitMQ，*其中包括了 交换机 和 队列。*
- 右侧 C 代表 消费者，也就是往 RabbitMQ 拿消息的程序。

##### 重要的概念：*虚拟主机，交换机，队列，和绑定*

- **虚拟主机**：一个虚拟主机持有一组交换机、队列和绑定。为什么需要多个虚拟主机呢？很简单，RabbitMQ当中，*用户只能在虚拟主机的粒度进行权限控制。* 因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”。
- **交换机**：*Exchange 用于转发消息，但是它不会做存储* ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。这里有一个比较重要的概念：**路由键** 。消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键。
- **绑定**：也就是交换机需要和队列相绑定，这其中如上图所示，是多对多的关系。

##### 交换机(Exchange)

交换机的功能主要是接收消息并且转发到绑定的队列，交换机不存储消息，在启用ack模式后，交换机找不到队列会返回错误。交换机有四种类型：Direct, topic, Headers and Fanout

- Direct：direct 类型的行为是”先匹配, 再投送”. 即在绑定时设定一个 **routing_key**, 消息的**routing_key** 匹配时, 才会被交换器投送到绑定的队列中去.

  ![rabbitmq-direct](/images/rabbitmq-direct.png)


- Topic：按规则转发消息（最灵活）

  ![rabbitmq-topic](/images/rabbitmq-topic.png)

- [Headers](http://codedestine.com/rabbitmq-headers-exchange/)：设置header attribute参数类型的交换机

- Fanout：转发消息到所有绑定队列

性能比较: [RabbitMQ三种Exchange模式(fanout,direct,topic)的性能比较](http://www.gaort.com/index.php/archives/366)

理解rabbitmq的概念 : [http://tryrabbitmq.com/](http://tryrabbitmq.com/)

![52232908396](/images/1522329083963.png)

![1523535278161](/images/1523535278161.png)

**示例代码**: [boot-in-action](https://github.com/silloy/boot-in-action)



**reference**: 

- [消息队列中间件调研文档](http://alibaba.github.io/RocketMQ-docs/document/openuser/mqvsmq.pdf)
- [几款消息中间的调研](http://blog.wentong.me/2016/01/message-queue-research/)
- [消息队列及常见消息队列介绍](https://cloud.tencent.com/developer/article/1006035)
- [Understanding When to use RabbitMQ or Apache Kafka](https://content.pivotal.io/blog/understanding-when-to-use-rabbitmq-or-apache-kafka)
- [高吞吐、高可用MQ对比分析](http://blog.zollty.com/b/archive/high-throughput-high-availability-MQ-comparative-analysis.html)
- [Kafka、RabbitMQ、RocketMQ消息中间件的对比 —— 消息发送性能](http://jm.taobao.org/2016/04/01/kafka-vs-rabbitmq-vs-rocketmq-message-send-performance/)
- [消息队列设计精要](https://tech.meituan.com/mq-design.html)
- [分布式开放消息系统(RocketMQ)的原理与实践](https://www.jianshu.com/p/453c6e7ff81c)
- [RabbitMQ详解](http://www.ityouknow.com/springboot/2016/11/30/springboot(%E5%85%AB)-RabbitMQ%E8%AF%A6%E8%A7%A3.html)
- [消息队列探秘-RabbitMQ消息队列介绍](https://www.jianshu.com/p/24f464f9161c)
- [RabbitMq延迟、重试队列及Spring Boot的黑科技](https://www.jianshu.com/p/35fbbdc9ca60)
- [rabbitmq可靠发送的自动重试机制](https://www.jianshu.com/p/6579e48d18ae)
- ​








