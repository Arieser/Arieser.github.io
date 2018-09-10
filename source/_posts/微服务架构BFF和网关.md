---
title: 微服务架构BFF和网关
date: 2018-09-05 16:01:01
---

BFF 架构图

![1536134559339](/images/1536134559339.png)

<!-- more -->

BFF(Backend for Frontend)也称聚合层或者适配层，上述架构从外到内依次为 端用户体验层->网关层->BFF层->微服务层，主要是讲内部复杂的微服务，适配成对各种不同的用户体验。网关专注解决跨横切面逻辑，包括路由、安全、监控和限流熔断等。

为提高系统的灵活性，在网关层和微服务层之间构建BFF层，这里主要使用GraphQL 构建BFF层。

GraphQL 特点：

1. 定义数据模型：按需获取

2. 数据分层：通过数据分层可以减少客户端请求次数

   ```json
   {
     user(id:1001) { // 第一层
       name,
       friends { // 第二层
         name,
         addr { // 第三层
           country,
           city
         }
       }
     }
   }
   ```

3. 强类型

   ```json
   const Meeting = new GraphQLObjectType({
     name: 'Meeting',
     fields: () => ({
       meetingId: {type: new GraphQLNonNull(GraphQLString)},
       meetingStatus: {type: new GraphQLNonNull(GraphQLString), defaultValue: ''}
     })
   })
   ```

   GraphQL 的类型系统定义了包括 Int, Float, String, Boolean, ID, Object, List, Non-Null 等数据类型。所以在开发过程中，利用强大的强类型检查，能够大大节省开发的时间，同时也很方便前后端进行调试。

4. 协议而非存储：GraphQL 本身并不直接提供后端存储的能力，不绑定任何的数据库或者存储引擎。

5. 无需版本化

   ```json
   const PhotoType = new GraphQLObjectType({
     name: 'Photo',
     fields: () => ({
       photoId: {type: new GraphQLNonNull(GraphQLID)},
       file: {
         type: new GraphQLNonNull(FileType),
         deprecationReason: 'FileModel should be removed after offline app code merged.',
         resolve: (parent) => {
           return parent.file
         }
       },
       fileId: {type: new GraphQLNonNull(GraphQLID)}
     })
   })
   ```

   GraphQL 服务端能够通过添加 deprecationReason，自动将某个字段标注为弃用状态。并且基于 GraphQL 高度的可扩展性，如果不需要某个数据，那么只需要使用新的字段或者结构即可，老的弃用字段给老的客户端提供服务，所有新的客户端使用新的字段获取相关信息。并且考虑到所有的 graphql 请求，都是按照 POST  `/graphql` 发送请求，所以在 GraphQL 中是无须进行版本化的。

GraphQL 与 Rest

1. **数据获取**：Rest 缺乏可扩展性，GraphQL能够按需获取；
2. **API调用**：REST针对每种资源的操作都是一个endpoint，GraphQL只需要一个endpoint；
3. **复杂数据请求**：REST对于嵌套的复杂数据需要多次调用，GraphQL 一次调用，减少网络开销；
4. **错误码处理**：REST能够精确返回HTTP错误码，GraphQL统一返回200，对错误信息进行包装；
5. **版本号**：REST通过v1/v2实现，Graph通过Schema扩展实现。



BFF 端技术栈

![1536247819506](/images/1536247819506.png)

[微服务下使用GraphQL构建BFF](https://juejin.im/entry/5abca4416fb9a028b92d38dd)中使用node作为BFF主要框架，因自己对node不太熟悉，会尽量采用java来实现相应的功能。GraphQL 使用 query 和 mutation 实现CQRS。