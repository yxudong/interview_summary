<a name="index">**目录：**</a><br>
&emsp;&emsp;&emsp;&emsp;&emsp;<a href="#0">select 语句完整的执行顺序</a><br>
&emsp;&emsp;&emsp;&emsp;&emsp;<a href="#1">Mysql 索引失效场景</a><br>
&emsp;&emsp;&emsp;&emsp;&emsp;<a href="#2">工作中遇到过有哪些死锁的场景？</a><br>
&emsp;&emsp;&emsp;&emsp;&emsp;<a href="#3">主从复制</a><br>
&emsp;&emsp;&emsp;&emsp;&emsp;<a href="#4">如何保证高可用</a><br>
&emsp;&emsp;&emsp;&emsp;&emsp;<a href="#5">两个索引列 or 会用到索引吗</a><br>
#### <a name="0">select 语句完整的执行顺序</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    1、from子句组装来自不同数据源的数据（先join在on）；
    2、where子句基于指定的条件对记录行进行筛选；
    3、group by子句将数据划分为多个分组；
    4、使用聚集函数进行计算；
    5、使用having子句筛选分组；
    6、计算所有的表达式；
    7、select 的字段；
    8、使用order by对结果集进行排序。

#### <a name="1">Mysql 索引失效场景</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        [索引失效的情况](https://learnku.com/articles/38889)

<p align='center'>
    <img src='./images/Mysql 索引失效场景.png'>
</p>

#### <a name="2">工作中遇到过有哪些死锁的场景？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

   todo
   replace into 高并发情况下有可能会造成死锁，可用 on duplicate key update 代替
   如果表中存在主键，另外存在唯一键，就可能会遇到这种死锁情况

#### <a name="3">主从复制</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

#### <a name="4">如何保证高可用</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

#### <a name="5">两个索引列 or 会用到索引吗</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
