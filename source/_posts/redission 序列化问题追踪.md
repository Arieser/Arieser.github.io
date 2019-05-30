---
title: redission 序列化问题追踪
date: 2019-05-28 17:26:43
tags:
  - redis
---



### 背景

项目原本是用jedis连接redis，但考虑到需要用redis锁，因此替换为方便快捷的**redisson**，但是使用redisson之后会报decode error，具体信息如下：

```java
2019-05-15 13:39:59.973 [redisson-netty-2-3] ERROR o.r.c.h.CommandDecoder [decodeCommand:203]     - Unable to decode data. channel: [id: 0x477c5ced, L:/192.168.4.94:57423 - R:10.10.10.43/10.10.10.43:6379], reply: ReplayingDecoderByteBuf(ridx=102, widx=102), command: (GET), params: [Geek:xxxxx:xxxx]
java.io.IOException: java.lang.NullPointerException
	at org.nustaq.serialization.FSTObjectInput.readObject(FSTObjectInput.java:247)
	at org.redisson.codec.FstCodec$1.decode(FstCodec.java:228)
	at org.redisson.client.handler.CommandDecoder.decode(CommandDecoder.java:368)
	at org.redisson.client.handler.CommandDecoder.decodeCommand(CommandDecoder.java:200)
	at org.redisson.client.handler.CommandDecoder.decode(CommandDecoder.java:140)
	at org.redisson.client.handler.CommandDecoder.decode(CommandDecoder.java:115)
	at io.netty.handler.codec.ByteToMessageDecoder.decodeRemovalReentryProtection(ByteToMessageDecoder.java:502)
	at io.netty.handler.codec.ReplayingDecoder.callDecode(ReplayingDecoder.java:366)
	at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:278)
```



<!-- more -->

### 测试代码

```java
RBucket ste = redissonClient.getBucket("Geek:add:ddd");
Object re = ste.get();
```

考虑可能是由于序列化产生的问题，查到[NullPointer 3.10.6][NullPointer 3.10.6]，设置codec为`StringCodec`，即

```java
redissonClient.getConfig().setCodec(new StringCodec());
```

但是并未解决问题，redisson仍然使用默认的`FstCodec`，通过idea强大的提示功能可以看到 getBucket接受一个codec参数

![idea.png](https://s2.ax1x.com/2019/05/15/ET2i0x.png)

修改代码为

```java
RBucket ste = redissonClient.getBucket("Geek:add:ddd", new StringCodec());
String re = ste.get();
```

完美解决

### 问题

为什么直接设置redisson config 不生效呢，一步步查源码 `RedissonObject#RedissonObject`

```
public RedissonObject(CommandAsyncExecutor commandExecutor, String name) {
        this(commandExecutor.getConnectionManager().getCodec(), commandExecutor, name);
}
```

可以看出redisson 默认从`ConnectionManager`里获取`codec`方式，继续看，以 `SingleConnectionManager` 为例，`SingleConnectionManager`是`MasterSlaveConnectionManager`的子类，具体的类图关系

![connectionManage.png](https://s2.ax1x.com/2019/05/15/ET4s1S.png)



config.java

```java
public Config(Config oldConf) {
        setExecutor(oldConf.getExecutor());
        if (oldConf.getCodec() == null) {
            // use it by default
            oldConf.setCodec(new FstCodec());
        }
......
    }
```

即检测到原有codec为空时，则设置为`FstCodec`

看一下 Redisson.java 配置关键部分代码

```java
protected Redisson(Config config) {
  this.config = config;
  Config configCopy = new Config(config);

  connectionManager = ConfigSupport.createConnectionManager(configCopy);
  evictionScheduler = new EvictionScheduler(connectionManager.getCommandExecutor());
  writeBehindService = new WriteBehindService(connectionManager.getCommandExecutor());
}
    
public static RedissonClient create(Config config) {
  Redisson redisson = new Redisson(config);
  if (config.isReferenceEnabled()) {
    redisson.enableRedissonReferenceSupport();
  }
  return redisson;
}
```

可以看出， config是在redisson初始化的时候传入的

因为我用的是`redisson-spring-boot-starter`，看一下这个starter里面，是如何初始化的，redisson starter 默认使用 spring-data-redis 配置。

```java
@Bean(destroyMethod = "shutdown")
    @ConditionalOnMissingBean(RedissonClient.class)
    public RedissonClient redisson() throws IOException {
        Config config = null;
       ....
        
        if (redissonProperties.getConfig() != null) {
            ....
        } else {
            config = new Config();
            String prefix = "redis://";
            Method method = ReflectionUtils.findMethod(RedisProperties.class, "isSsl");
            if (method != null && (Boolean)ReflectionUtils.invokeMethod(method, redisProperties)) {
                prefix = "rediss://";
            }
            
            config.useSingleServer()
                .setAddress(prefix + redisProperties.getHost() + ":" + redisProperties.getPort())
                .setConnectTimeout(timeout)
                .setDatabase(redisProperties.getDatabase())
                .setPassword(redisProperties.getPassword());
        }
        
   return Redisson.create(config);
```

回到一开始的问题，直接设置redisson codec为什么不生效？仔细以上分析可以知道，redisson统一设置codec主要是通过初始化的时候传入**ConnectionManager**使 codec生效，而通过 `redissonClient.getConfig().setCodec(...)`的方式并不能改变**ConnectionManager**中的编码方式。

结论：

1. 如果想自定义codec，需要自己初始化redissonClient[调用Redisson.create(config)]， 或者重写redisson-starter
2. 在定制化程度不高时，可直接使用默认codec，或者把特定的codec传入方法体内





**Reference**

[NullPointer 3.10.6]: https://github.com/redisson/redisson/issues/2032	"Issue"