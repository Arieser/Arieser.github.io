---
title: 使用RabbitMQ实现AMQP和MQTT的协议转换
date: 2018-04-13 09:40:00
tags:
  - amqp
  - mqtt
---



基于LBS的业务系统，前台使用MQTT协议推送到后台，后台通过RabbitMQ实现协议的转换，以实现负载消费的功能。主要的框架如图

![1523585173759](/images/1523585173759.png)

<!--more-->

准备工作：rabbitmq安装以及 rabbitmq_mqtt 插件启用

MQTT client向`/drivers/1`push消息，AMQP client 使用 topic 模式接受消息， exchange name为固定值 `amq.topic`，routingKey 为 MQTT 发送使用的Topic，注意`/`需要替换为`.`，queue name 可以任意指定，绑定相同queue的customer可以实现负载消费的功能。

![1523585612511](/images/1523585612511.png)

Demo示例(以java为例)：

RabbitConfig

```java
@Component
public class TopicRabbitConfig {

    public static final String message = "location.broker";

    @Bean
    public Queue queueMessage() {
        return new Queue(message);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("amq.topic", true, false);
    }

    //綁定队列 queueMessages() 到 topicExchange 交换机,路由键只接受完全匹配 topic.message 的队列接受者可以收到消息, # 为通配符模式
    @Bean
    Binding bindingExchangeMessage(Queue queueMessage, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessage).to(exchange).with('.drivers.#');
    }

}
```



RabbitListener:

```java
@Component
@RabbitListener(queues = "location.broker")
public class TopicReciver {

    @RabbitHandler
    public void process(byte[] hello) {
        System.out.println(new String(hello));
    }
}
```

使用java模拟的mqtt client 发送的消息为byte[]， 因此需要使用`byte[]`接收消息内容。

**References**

- [Uniting AMQP and MQTT Message Brokering with RabbitMQ](https://blogs.sap.com/2016/02/21/uniting-amqp-and-mqtt-message-brokering-with-rabbitmq/)
- [使用rabbitmq做为mqtt服务器，整合spring做推送后台](https://my.oschina.net/u/1047640/blog/819418)