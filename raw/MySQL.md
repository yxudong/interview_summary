# select 语句完整的执行顺序

    1、from子句组装来自不同数据源的数据（先join在on）；
    2、where子句基于指定的条件对记录行进行筛选；
    3、group by子句将数据划分为多个分组；
    4、使用聚集函数进行计算；
    5、使用having子句筛选分组；
    6、计算所有的表达式；
    7、select 的字段；
    8、使用order by对结果集进行排序。

# Mysql 索引失效场景

    参考：
        [索引失效的情况](https://learnku.com/articles/38889)

<p align='center'>
    <img src='./images/Mysql 索引失效场景.png'>
</p>

# 工作中遇到过有哪些死锁的场景？

   todo
   replace into 高并发情况下有可能会造成死锁，可用 on duplicate key update 代替
   如果表中存在主键，另外存在唯一键，就可能会遇到这种死锁情况

# 主从复制

    todo
    https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/mysql-read-write-separation.md

# 如何保证高可用

    todo

# 两个索引列 or 会用到索引吗

    todo

# 索引为什么使用 b+ 树，不是 b 树？

    todo
    https://segmentfault.com/a/1190000021488885
    https://zhuanlan.zhihu.com/p/345414925#:~:text=B%2B树与B树,完成查询范围的查找。

# select * from A where x=1 and y > 1 order by z；让你设计索引什么设计？为啥这样？还有如果按照你设计的索引，mysql他的执行查询的树状图是怎样的？可否画一下？

    todo

# 大表 DDL 方案

    todo
    https://www.modb.pro/db/40251
    https://zhuanlan.zhihu.com/p/115277009

# 分库分表的区别

    todo

# 分区表和手动分表的区别，为什么不使用分区表

    todo
