---
title: Spring Data MongoDB 查询语法
date: 2019-08-21 13:26:43
tags:
  - spring data
---



<!-- more -->

1. Query 和 Criteria 查询

   ```java
   Query query = new Query();
   query.addCriteria(Criteria.where("name").is("Eric"));
   List<User> users = mongoTemplate.find(query, User.class);
   ```

   支持的查询方法：is, regex, lt, gt, pageable, sort

2. 生成query方法

   - findByX

     ```java
     List<User> findByName(String name);
     ```

   - startinggWith and endingWith

     ```java
     List<User> findByNameStartingWith(String regexp);
     List<User> findByNameEndingWith(String regexp);
     ```

   - between

     ```java
     List<User> findByAgeBetween(int ageGT, int ageLT);
     ```

   - like and orderBy

     ```java
     List<User> users = userRepository.findByNameLikeOrderByAgeAsc("A");
     ```

3. JSON Query methods : @Query

   ```java
   @Query("{ 'name' : ?0 }")
   List<User> findUsersByName(String name);
   ```

   支持的查询方法： $regex,  $gt,  $lt

4. QueryDSL Queries

   4.1 maven

   ```xml
   <dependency>
       <groupId>com.mysema.querydsl</groupId>
       <artifactId>querydsl-mongodb</artifactId>
       <version>3.6.6</version>
   </dependency>
   <dependency>
       <groupId>com.mysema.querydsl</groupId>
       <artifactId>querydsl-apt</artifactId>
       <version>3.6.6</version>
   </dependency>
   
   
   <plugin>    
       <groupId>com.mysema.maven</groupId>
       <artifactId>apt-maven-plugin</artifactId>
       <version>1.1.3</version>
       <executions>
           <execution>
               <goals>
                   <goal>process</goal>
               </goals>
               <configuration>
                   <outputDirectory>target/generated-sources/java</outputDirectory>
                   <processor>
                 org.springframework.data.mongodb.repository.support.MongoAnnotationProcessor
                   </processor>
               </configuration>
           </execution>
        </executions>
   </plugin>
   ```

   4.2 class

   ```java
   @QueryEntity
   @Document
   public class User {
     
       @Id
       private String id;
       private String name;
       private Integer age;
     
       // standard getters and setters
   }
   ```

5. Implement **QueryDslPredicateExecutor**

   ```java
   QUser qUser = new QUser("user");
   Predicate predicate = qUser.name.eq("Eric");
   List<User> users = (List<User>) userRepository.findAll(predicate);
   ```

   支持的查询方法：is,startinggWith and endingWith, between


​    