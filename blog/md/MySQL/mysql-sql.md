---
title: MySQL 中常用 SQL 的优化
date: 2019-2-13 15:16:46
categories: [开发,数据库]
tags: [MySQL]
---

## 前言
之前介绍了 MySQL 中怎么样通过索引来优化查询。日常开发中，除了使用查询外，我们还会使用一些其他的常用 SQL，比如 INSERT、GROUP BY 等。对于这些 SQL 语句，我们该怎么样进行优化呢？接下来将针对这些 SQL 语句介绍一些优化的方法。

## 大批量插入数据
当用 load 命令导入数据的时候，适当的设置可以提高导入的速度。

对于 MyISAM 存储引擎的表，可以通过以下方式快速的导入大量的数据。
```
ALTER TABLE tbl_name DISABLE KEYS;
loading the data
ALTER TABLE tbl_name ENABLE KEYS;
```
`DISABLE KEYS` 和 `ENABLE KEYS` 用来打开或者关闭 MyISAM 表非唯一索引的更新。在导入大量的数据到一个非空的 MyISAM 表时，通过设置这两个命令，可以提高导入的效率。对于导入大量数据到一个空的 MyISAM 表，默认就是先导入数据然后才创建索引的，所以不用进行设置。

下面例子中，用 LOAD 语句导入数据耗时 115.12 秒：
```
mysql> load data infile '/home/mysql/film_test.txt' into table film_test2;
Query OK,529056 rows affected(1 min 55.12 sec)
Records: 529056  Deleted: 0  Skipped: 0  Warning: 0
```
而用 `alter table tbl_name disable keys` 方式总耗时 6.34+12.25 = 18.59 秒，提高了 6 倍多。
```
mysql> alter table film_test2 disable keys;
Query OK,0 rows affected(0.00 sec)

mysql> load data infile '/home/mysql/film_test.txt' into table film_test2;
Query OK,529056 rows affected(6.34 sec)
Records: 529056  Deleted: 0  Skipped: 0  Warnings: 0

mysql> alter table film_test2 enable keys;
Query OK,0 rows affected(12.25 sec)
```
上面是对 MyISAM 表进行数据导入时的优化措施，对于 InnoDB 类型的表，这种方式并不能提高导入数据的效率，可以有以下几种方式提高 InnoDB 表的导入效率。

（1）因为 InnoDB 类型的表是按照主键的顺序保存的，所以将导入的数据按照主键的顺序排列，可以有效地提高导入数据的效率。

例如，下面文本 film_test3.txt 是按表 film_tst4 的主键存储的，那么导入的时候共耗时 27.92 秒。
```
mysql> load data infile '/home/mysql/film_test3.txt' into table film_test4;
Query OK,1587168 rows affected(22.92 sec)
Records: 1587168  Deleted: 0  Skipped: 0  Warning: 0
```
而下面的 film_test4.txt 是没有任何顺序的文本，那么导入的时候共耗时 31.16 秒。
```
mysql> load data infile '/home/mysql/film_test4.txt' into table film_test4;
Query OK,1587168 rows affected(31.16 sec)
Records: 1587168  Deleted: 0  Skipped: 0  Warning: 0
```
从上面的例子可以看出当被导入的文件按表主键顺序存储的时候比不按主键顺序存储的时候快 1.12 倍。

（2）在导入数据前执行 `SET UNIQUE_CHECKS=0`，关闭唯一性校验，在导入结束后执行 `SET UNIQUE_CHECK=1`，恢复唯一性校验，可以提高导入的效率。

例如，当 UNIQUE_CHECKS=1 时：
```
mysql> load data infile '/home/mysql/film_test3.txt' into table film_test4;
Query OK,1587168 rows affected(22.92 sec)
Records: 1587168  Deleted: 0  Skipped: 0  Warning: 0
```
当 SET UNIQUE_CHECKS=0 时：
```
mysql> load data infile '/home/mysql/film_test3.txt' into table film_test4;
Query OK,1587168 rows affected(19.92 sec)
Records: 1587168  Deleted: 0  Skipped: 0  Warning: 0
```
可见比 UNIQUE_CHECKS=0 的时候比 SET UNIQUE_CHECKS=1 的时候要快一些。

（3）如果应用使用自动提交的方式，建议在导入前执行 `SET AUTOCOMMIT=0`，关闭自动提交，导入结束后再执行 `SET AUTOCOMMIT=1`，打开自动提交，也可以提高导入的效率。

例如，当 AUTOCOMMIT=1 时：
```
mysql> load data infile '/home/mysql/film_test3.txt' into table film_test4;
Query OK,1587168 rows affected(22.92 sec)
Records: 1587168  Deleted: 0  Skipped: 0  Warning: 0
```
当 AUTOCOMMIT=0 时：
```
mysql> load data infile '/home/mysql/film_test3.txt' into table film_test4;
Query OK,1587168 rows affected(20.87 sec)
Records: 1587168  Deleted: 0  Skipped: 0  Warning: 0
```
对比一下可以知道，当 AUTOCOMMIT=0 时比 AUTOCOMMIT=1 时导入数据要快一些。

## 优化 INSERT 语句
当进行数据 INSERT 的时候，可以考虑采用以下几种优化方式。

- 如果同时从同一客户插入很多行，尽量使用多个值表的 INSERT 语句，这种方式将大大缩减客户端与数据库之间的连接、关闭等消耗，使得效率比分开执行的单个 INSERT 语句快（在一些情况中几倍）。下面是一次插入多值的一个例子：
```
insert into test values(1,2),(1,3),(1,4)...
```
- 如果从不同客户插入很多行，能通过使用 `INSERT DELAYED` 语句得到更高的速度。DELAYED 的含义是让 INSERT 语句马上执行，其实数据都被放在内存的队列中，并没有真正写入磁盘，这比每条语句分别插入要快得多；`LOW_PRIORITY` 刚好相反，在所有其他用户对表的读写完后才进行插入；
- 将索引文件和数据文件分在不同的磁盘上存放（利用建表中的选项）；
- 如果进行批量插入，可以增加 `bulk_insert_buffer_size` 变量值的方法来提高速度，但是，这只能对 MyISAM 表使用；
- 当从一个文本文件装载一个表时，使用 `LOAD DATA INFILE`。这通常比使用很多 INSERT 语句快 20 倍。

## 优化 GROUP BY 语句
默认情况下，MySQL 对所有 GROUP BY col1，col2...的字段进行排序。这与在查询中指定 ORDER BY col1，col2...类似。因此，如果显式包括一个包含相同的列的 ORDER BY 子句，则对 MySQL 的实际执行性能没有什么影响。

如果查询包括 GROUP BY 但用户想要避免排序结果的消耗，则可以指定 ORDER BY NULL 禁止排序，如下面的例子：
```
mysql> explain select id, sum(moneys) from sales2 group by id\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: sales2
               type: ALL
      possible_keys: NULL
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using temporary; Using filesort
    1 row in set(0.00 sec)

mysql> explain select id, sum(moneys) from sales2 group by id order by null\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: sales2
               type: ALL
      possible_keys: NULL
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using temporary
```
从上面的例子可以看出第一个 SQL 语句需要进行“filesort”，而第二个 SQL 由于 ORDER BY NULL 不需要进行“filesort”，而 filesort 往往非常耗费时间。

## 优化 ORDER BY 语句
在某些情况中，MySQL 可以使用一个索引来满足 ORDER BY 子句，而不需要额外的排序。WHERE 条件和 ORDER BY 使用相同的索引，并且 ORDER BY 的顺序和索引顺序相同，并且 ORDER BY 的字段都是升序或者都是降序。

例如，下列 SQL 可以使用索引：
```
SELECT * FROM t1 ORDER BY key_part1,key_part2,...;
SELECT * FROM t1 WHERE key_part1=1 ORDER BY key_part1 DESC, key_part2 DESC;
SELECT * FROM t1 ORDER BY key_part1 DESC, key_part2 DESC;
```
但是在以下几种情况下则不使用索引：
```
SELECT * FROM t1 ORDER BY key_part1 DESC, key_part2 ASC;  --order by 的字段混合 ASC 和 DESC
SELECT * FROM t1 WHERE key2=constant ORDER BY key1;  --用于查询行的关键字与 ORDER BY 中所使用的不相同
SELECT * FROM t1 ORDER BY key1, key2;  --对不同的关键字使用 ORDER BY
```

## 优化嵌套查询
MySQL 4.1 开始支持 SQL 的子查询。这个技术可以使用 SELECT 语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性地完成很多逻辑上需要多个步骤才能完成的 SQL 操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询可以被更有效率的连接（JOIN）替代。

在下面的例子中，要从 sales2 表中找到那些在 company2 表中不存在的所有公司的信息：
```
mysql> explain select * from sales2 where company_id not in (select id from company2)\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: PRIMARY
              table: sales2
               type: ALL
      possible_keys: NULL
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using where
    **************************** 2. row *************************************
                 id: 2
        select_type: DEPENDENT SUBQUERY
              table: company2
               type: index_subquery
      possible_keys: ind_company2_id
                key: ind_company2_id
            key_len: 5
                ref: func
               rows: 2
              Extra: Using index

   2 rows in set(0.00 sec)
```
如果使用连接（JOIN）来完成这个查询工作，速度将会快很多。尤其是当 company2 表中对 id 建有索引的话，性能将会更好，具体查询如下：
```
mysql> explain select * from sales2 left join company2 on sales2.company_id = company2.id where sales2.company_id is null\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: sales2
               type: ref
      possible_keys: ind_sales2_companyid_moneys
                key: ind_sales2_companyid_moneys
            key_len: 5
                ref: const
               rows: 1
              Extra: Using where
    **************************** 2. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: ref
      possible_keys: ind_company2_id
                key: ind_company2_id
            key_len: 5
                ref: sakila.sales2.company_id
               rows: 1
              Extra: 
   2 rows in set(0.00 sec)
```
从执行计划中可以明显看出查询扫描的记录范围和使用索引的情况都有了很大的改善。

**连接（JOIN）之所以更有效率一些，是因为 MySQL 不需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作。**

## 优化 OR 条件
**对于含有 OR 的查询子句，如果要利用索引，则 OR 之间的每个条件列都必须用到索引；如果没有索引，则应该考虑增加索引。**

例如，首先使用 `show index` 命令查看表 sales2 的索引，可知它有 3 个索引，在 id、year 两个字段上分别有 1 个独立的索引，在 company_id 和 year 字段上有 1 个复合索引。
```
mysql> show index from sales2\G;
    **************************** 1. row *************************************
              Table: sales2
         Non_unique: 1
           Key_name: ind_sales2_id
       Seq_in_index: 1
        Column_name: id
          Collation: A
        Cardinality: 1000
           Sub_part: NULL
             Packed: NULL
               Null: YES
         Index_type: BTREE
            Comment:
    **************************** 2. row *************************************
              Table: sales2
         Non_unique: 1
           Key_name: ind_sales2_year
       Seq_in_index: 1
        Column_name: year
          Collation: A
        Cardinality: 250
           Sub_part: NULL
             Packed: NULL
               Null: YES
         Index_type: BTREE
            Comment:
    **************************** 3. row *************************************
              Table: sales2
         Non_unique: 1
           Key_name: ind_sales2_companyid_moneys
       Seq_in_index: 1
        Column_name: company_id
          Collation: A
        Cardinality: 1000
           Sub_part: NULL
             Packed: NULL
               Null: YES
         Index_type: BTREE
            Comment:
    **************************** 4. row *************************************
              Table: sales2
         Non_unique: 1
           Key_name: ind_sales2_companyid_moneys
       Seq_in_index: 2
        Column_name: year
          Collation: A
        Cardinality: 1000
           Sub_part: NULL
             Packed: NULL
               Null: YES
         Index_type: BTREE
            Comment:
    4 rows in set(0.00 sec)
```
然后在两个独立索引上面做 OR 操作，具体如下：
```
mysql> explain select * from sales2 where id = 2 or year = 1998\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: sales2
               type: index_merge
      possible_keys: ind_sales2_id,ind_sales2_year
                key: ind_sales2_id,ind_sales2_year
            key_len: 5,2
                ref: NULL
               rows: 2
              Extra: Using union(ind_sales2_id,ind_sales2_year); Using where
    1 row in set(0.00 sec)
```
可以发现查询正确的用到了索引，并且从执行计划的描述中，**发现 MySQL 在处理含有 OR 子句的查询时，实际是对 OR 的各个字段分别查询后的结果进行了 UNION。**

但是当在建有复合索引的列 company_id 和 moneys 上面做 OR 操作的时候，却不能用到索引，具体结果如下：
```
mysql> explain select * from sales2 where company_id = 3 or moneys = 100\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: sales2
               type: ALL
      possible_keys: ind_sales2_companyid_moneys
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using where
    1 row in set(0.00 sec)
```

> 本文大多摘自《深入浅出MySQL》。