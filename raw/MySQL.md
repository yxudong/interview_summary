#### select 语句完整的执行顺序

    1、from子句组装来自不同数据源的数据（先join在on）；
    2、where子句基于指定的条件对记录行进行筛选；
    3、group by子句将数据划分为多个分组；
    4、使用聚集函数进行计算；
    5、使用having子句筛选分组；
    6、计算所有的表达式；
    7、select 的字段；
    8、使用order by对结果集进行排序。

#### Mysql 索引

#### Mysql 索引失效场景

    参考：
        [索引失效的情况](https://learnku.com/articles/38889)

<p align='center'>
    <img src='./images/Mysql 索引失效场景.png'>
</p>

#### Mysql 事务 MVCC 机制

#### Mysql 锁

#### 主从复制

#### 如何保证高可用












#### 工作中遇到过有哪些死锁的场景？

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



