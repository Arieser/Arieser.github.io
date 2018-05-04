---
title: MongoDB CURD操作及在Spring环境中的使用
date: 2018-04-09 13:50:00
tags:
  - MongoDB
---

MongoDB作为高性能，开源，无模式的文档型数据库，简要记录一下其CURD操作及其在spring boot 中的使用。

| MongoDB          | RDBMS            |
| ---------------- | ---------------- |
| 数据库(database) | 数据库(database) |
| 集合(collection) | 表(table)        |
| 文档(document)   | 行(row)          |

<!--more-->

**环境**:

- MondoDB:  3.6.3 windows
- GUI: Studio 3T




#### 基本CURD操作

[MongoDB 中文文档](http://www.mongoing.com/docs/)

[The MongoDB 3.6 Manual](https://docs.mongodb.com/manual/)

[MongoDB手册](https://mongodb-documentation.readthedocs.io/en/latest/)


#### 使用MongoDB操作地理数据(GeoJSON)

地理空间操作符

![1523253863147](/images/1523253863147.png)



数据格式demo:
```
wget https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/restaurants.json
```

```json
{
location: {
    type: "Point",
    coordinates: [-73.856077, 40.848447]
},
name: "Morris Park Bake Shop"
}
```

1. 创建2dsphere索引

   `db.restaurants.createIndex({ location: "2dsphere" })`

2. 围栏算法，返回单点一定距离内的位置

   ```
   db.restaurants.find({ location:
   { $geoWithin:
   { $centerSphere: [ [ -73.93414657, 40.82302903 ], 5 / 3963.2 ] } } })
   ```

   其中，`location`是一个包含地理坐标数据、需要匹配的字段。
   `$centerSphere`的第二个参数接收弧度制的半径，因此我们必须使用它来除以地球的半径（单位为mile）。查阅[相关文档](https://docs.mongodb.com/manual/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/)了解更多关于距离单位之间的换算。

   **使用 `$nearSphere`进行排序**

   ```
   db.restaurants.find({ location: { $nearSphere: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] }, $maxDistance: 5 * 1000 } } })
   ```

   `$maxDistance` 参数：单位为米


reference:  http://www.mongoing.com/mongodb-geo-index-1/



#### SpringBoot中MongoDB相关注解

- **@Id**

  主键，不可重复，不建议自己设置，使用自动生成主键插入效率远高于自己设置主键。

  性能对比(每1000条数据插入): [MongoDB与MySQL的插入、查询性能测试](https://blog.csdn.net/tianyaleixiaowu/article/details/73504335)

  ![1523254943332](/images/1523254943332.png)

- **@Document**

  标注在实体类上，类似于hibernate的entity注解，标明由mongo来维护该表

- **@Indexed**

  声明该字段需要加索引，加索引后以该字段为条件检索将大大提高速度。使用方法待查

- **@CompoundIndex**

  复合索引

  ```java
  @Document
  @CompoundIndexes({
      @CompoundIndex(name = "age_idx", def = "{'lastName': 1, 'age': -1}")
  })
  public class Person<T extends Address> {
  }
  ```

- **@Field**

  指定参数在Mongo中的列名，默认以参数名称为列名

- **@Transient**

  该注解标注的，将不会被录入到数据库中，只作为普通的java bean属性

- **@DBRef**

  ​