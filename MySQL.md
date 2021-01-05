# Mysql



**1. innoDB 是否支持哈希索引？**

   **innodb 支持自适应哈希索引，不能手动地创建哈希索引，innodb 会进行 self-tuning (自优化)。如果判断建立哈希索引，有助于提升查询性能**

    分析一个查询的过程。
    以表 employee 为例，字段有 id(主键),name,idcode(身份证号)等。建立在 idcode 字段上建立二级索引。
    如果根据 idcode 查询用户信息，在二级索引上查出主键；再根据主键回表查询聚集索引，才能获取数据。

    记录定位的寻路路径（Search Path）很长，即便是聚集索引查询也是需要几次记录定位。innodb 是这样解决这个问题的。

    在 MySQL 运行的过程中，如果 InnoDB 发现，有很多 SQL 存在这类很长的寻路，并且有很多SQL会命中相同的页面（page），InnoDB 会在自己的内存缓冲区里，
    开辟一块区域，建立自适应哈希所有 AHI，以加速查询。

**2. 有哪些死锁的场景？**

   replace into 高并发情况下有可能会造成死锁，可用 on duplicate key update 代替
   如果表中存在主键，另外存在唯一键，就可能会遇到这种死锁情况





### 参考文档

https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md

- 一、索引
  - [B+ Tree 原理](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#b-tree-原理)
  - [MySQL 索引](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#mysql-索引)
  - [索引优化](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#索引优化)
  - [索引的优点](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#索引的优点)
  - [索引的使用条件](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#索引的使用条件)
- 二、查询性能优化
  - [使用 Explain 进行分析](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#使用-explain-进行分析)
  - [优化数据访问](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#优化数据访问)
  - [重构查询方式](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#重构查询方式)
- 三、存储引擎
  - [InnoDB](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#innodb)
  - [MyISAM](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#myisam)
  - [比较](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#比较)
- 四、数据类型
  - [整型](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#整型)
  - [浮点数](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#浮点数)
  - [字符串](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#字符串)
  - [时间和日期](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#时间和日期)
- 五、切分
  - [水平切分](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#水平切分)
  - [垂直切分](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#垂直切分)
  - [Sharding 策略](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#sharding-策略)
  - [Sharding 存在的问题](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#sharding-存在的问题)
- 六、复制
  - [主从复制](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#主从复制)
  - [读写分离](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#读写分离)



