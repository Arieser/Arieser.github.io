---
title: SpringBoot-Mybatis通用mapper使用
date: 2018-06-15 13:36:43
tags: 
  - spring boot
---

mybatis是一个很好用的工具，但是编写mapper是一件很麻烦的事，自mybatis 3.0开始可以使用注解的方式，极大的简化了xml的编写量，本地想看看mybatis源码，自己扩展写一个工具，在阅读源码过程中发现一个通用mapper的工具包，感觉不用重复造轮子了，简要记录一下spring boot整合通用mapper的使用。

![1529042422800](/images/1529042422800.png)

<!--more-->

0. 确保可以正常使用mybatis

1. pom引入依赖包，starter需要配合@Mapper注解使用，这里采用这种方式，或者使用`@MapperScan`注解，`@tk.mybatis.spring.annotation.MapperScan(basePackages = "扫描包")`配合原生mapper使用。

   ```java
   <dependency>
     <groupId>tk.mybatis</groupId>
     <artifactId>mapper-spring-boot-starter</artifactId>
     <version>{version}</version>
   </dependency>
   ```

   我使用的版本是2.0.[2]()

2. Mybatis 扫描配置（Deprecated, spring 自动配置）

   ```java
   @Configuration
   //TODO 注意，由于MapperScannerConfigurer执行的比较早，所以必须有下面的注解
   @AutoConfigureAfter(MybatisAutoConfiguration.class)
   public class MyBatisMapperScannerConfig {
       @Bean
       public MapperScannerConfigurer mapperScannerConfigurer() {
           MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
           mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactory");
           mapperScannerConfigurer.setBasePackage("org.springboot.sample.mapper");
           Properties properties = new Properties();
           // 这里要特别注意，不要把MyMapper放到 basePackage 中，也就是不能同其他Mapper一样被扫描到。 
           properties.setProperty("mappers", MyMapper.class.getName());
           properties.setProperty("notEmpty", "false");
           properties.setProperty("IDENTITY", "MYSQL");
           mapperScannerConfigurer.setProperties(properties);
           return mapperScannerConfigurer;
       }
   }
   ```

3. 新建BaseMapper类，该类不能被当做普通Mapper一样被扫描 ，不加@Mapper注解，或者放在不同文件夹

   ```java
   package com.zj.mapper;
   
   import tk.mybatis.mapper.common.Mapper;
   import tk.mybatis.mapper.common.MySqlMapper;
   
   public interface BaseMapper<T> extends Mapper<T>, MySqlMapper<T> {
   }
   ```

4. 业务处理dao层，扩展BaseMapper

   ```java
   package com.zj.mapper;
   
   import com.zj.model.OrderInfo;
   import org.apache.ibatis.annotations.Mapper;
   
   @Mapper
   public interface OrderInfoMapper extends BaseMapper<OrderInfo> {}
   ```

5. 其他和使用普通mybatis一致，service层部分代码

   ```java
   orderInfoMapper.insertSelective(info);
   OrderInfo info = orderInfoMapper.selectByPrimaryKey(id);
   ```

   通用mapper提供常用的一些操作方法: deleteByPrimaryKey, insert, insertSelective, selectByPrimaryKey, updateByPrimaryKeySelective, updateByPrimaryKey, insertList等很多方法，需要你进一步探索😀😀

6. 主键id问题

   当使用insert，insertSelective等方法时，希望返回由数据库产生的逐渐，需要在实体类上增加注解

   ```java
   @Id
   @GeneratedValue(generator = "JDBC")
   private Long orderInfoId;
   ```

   generator="JDBC"表示 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键 ，适用于MySQL，SQL Server等的自增主键。

   或者：

   ```java
   @Id
   @KeySql(useGeneratedKeys = true)
   private Long id;
   ```

7. 如果实体字段和数据库字段不一致，可以使用`@Column`注解，其他注解 参见[注解][annotation]

   ```java
    @Column(name="SCORE_SUM")
    private String sumScore;
   ```

8. MBG生成参见https://github.com/abel533/Mapper/wiki/4.1.mappergenerator，demo见 git@github.com:silloy/mybatis-generator.git

9. 更多细节参见wiki

   

   

   通用Mapper极大的简化了xml文件的编写，但仍需要少许xml文件，有待进一步优化。同时因为这是一个个人项目，使用不太熟悉不建议使用。

     
[annotation]: https://github.com/abel533/Mapper/wiki/2.2-mapping

**References**

http://www.mybatis.tk/

https://github.com/abel533/Mapper/wiki

https://github.com/abel533/Mapper

https://blog.csdn.net/qq_19260029/article/details/78010369