---
title: MySQL 中的索引问题
date: 2019-2-13 13:29:01
categories: [开发,数据库]
tags: [MySQL]
---

## 前言
索引是数据库优化中最常用也是最重要的手段之一，通过索引通常可以帮助用户解决大多数的 SQL 性能问题。接下来将对 MySQL 中的索引的分类、存储、使用方法做详细的介绍。

## 索引的存储分类
**MyISAM 存储引擎的表的数据和索引是自动分开存储的，各自是独立的一个文件；InnoDB 存储引擎的表的数据和索引是存储在同一个表空间里面，但可以有多个文件组成。**

MySQL 中索引的存储类型目前只有两种（BTREE 和 HASH），具体和表的存储引擎相关：MyISAM 和 InnoDB 存储引擎都只支持 BTREE 索引；MEMORY/HEAP 存储引擎可以支持 HASH 和 BTREE 索引。

MySQL 目前不支持函数索引，但是能对列的前面某一部分进索引，例如 name 字段，可以只取 name 的前 4 个字符进行索引，这个特性可以大大缩小索引文件的大小，用户在设计表结构的时候也可以对文本列根据此特性进行灵活设计。下面是创建前缀索引的一个例子：
```
mysql> create index ind_company2_name on company2(name(4));
Query OK,1000 rows affected(0.03 sec)
Records: 1000  Duplicates: 0   Warnings: 0
```

## 索引的使用
**索引用于快速找出在某个列中有一特定值的行。对相关列使用索引是提高 SELECT 操作性能的最佳途径。**

**查询要使用索引最主要的条件是查询条件中需要使用索引关键字，如果是多列索引，那么只有查询条件使用了多列关键字最左边的前缀时，才可以使用索引，否则将不能使用索引。**

### 使用索引
在 MySQL 中，下列几种情况下有可能使用到索引。

（1）对于创建的多列索引，只要查询的条件中用到了最左边的列，索引一般就会被使用，举例说明如下。

**首先按 company_id，moneys 的顺序创建一个复合索引，具体如下：**
```
mysql> create index ind_sales2_companyid_moneys on sales2(company_id, moneys);
Query OK,1000 rows affected(0.03 sec)
Records: 1000  Duplicates: 0  Warnings: 0
```
**然后按 company_id 进行表查询，具体如下：**
```
mysql> explain select * from sales2 where company_id = 2006\G;
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
    1 row in set(0.00 sec)
```
**可以发现即便 where 条件中不是用的 company_id 与 moneys 的组合条件，索引仍然能用到，这就是索引的前缀特性。但是如果只按 moneys 条件查询表，那么索引就不会被用到**，具体如下：
```
mysql> explain select * from sales2 where moneys = 1\G;
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
              Extra: Using where
    1 row in set(0.00 sec)
```
（2）对于使用 like 的查询，后面如果是常量并且只有 `%` 号不在第一个字符，索引才可能会被使用，来看下面两个执行计划：
```
mysql> explain select * from company2 where name like '%3'\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: ALL
      possible_keys: NULL
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using where
    1 row in set(0.00 sec)

mysql> explain select * from company2 where name like '3%'\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: range
      possible_keys: ind_company2_name
                key: ind_company2_name
            key_len: 11
                ref: NULL
               rows: 103
              Extra: Using where
    1 row in set(0.00 sec)
```
可以发现第一个例子没有使用索引，而第二例子就能够使用索引，区别就在于“%”的位置不同，前者把“%”放到第一位就不能用到索引，而后者没有放到第一位就使用了索引。

**另外，如果 like 后面跟的是一个列的名字，那么索引也不会被使用。**

（3）如果对大的文本进行搜索，使用全文索引而不用使用 like '%...%'。

（4）如果列名是索引，使用 column_name is null 将使用索引。如下例中查询 name 为 null 的记录就用到了索引：
```
mysql> explain select * from company2 where name is null\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: ref
      possible_keys: ind_company2_name
                key: ind_company2_name
            key_len: 11
                ref: const
               rows: 1
              Extra: Using where
    1 row in set(0.00 sec)
```

### 存在索引但不使用索引
在下列情况下，虽然存在索引，但是 MySQL 并不会使用相应的索引。

（1）如果 MySQL 估计使用索引比全表扫描更慢，则不使用索引。例如如果列 key_part1 均匀分布在 1 和 100 之间，下列查询中使用索引就不是很好：
```
SELECT * FROM table_name where key_part1 > 1 and key_part1 < 90;
```

（2）如果使用 MEMORY/HEAP 表并且 where 条件中不使用“=”进行索引列，那么不会用到索引。heap 表只有在“=”的条件下才会使用索引。

（3）用 or 分割开的条件，如果 or 前的条件中的列有索引，而后面的列中没有索引，那么涉及到的索引都不会被用到，例如：
```
mysql> show index from sales\G;
    **************************** 1. row *************************************
              Table: sales
         Non_unique: 1
           Key_name: ind_sales_year
       Seq_in_index: 1
        Column_name: year
          Collation: A
        Cardinality: NULL
           Sub_part: NULL
             Packed: NULL
               Null:
         Index_type: BTREE
            Comment:
    1 row in set(0.00 sec)
```
从上面可以发现只有 year 列上面有索引，来看如下的执行计划：
```
mysql> explain select * from sales where year = 2001 or country = 'China'\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: sales
               type: ALL
      possible_keys: ind_sales_year
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 12
              Extra: Using where
    1 row in set(0.00 sec)
```
可见虽然在 year 这个列上存在索引 `ind_sales_year`，但是这个 SQL 语句并没有用到这个索引，原因就是 or 中有一个条件中的列没有索引。

（4）如果不是索引列的第一部分，如下例子：
```
mysql> explain select * from sales2 where moneys = 1\G;
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
              Extra: Using where
    1 row in set(0.00 sec)
```
可见虽然在 money 上面建有复合索引，但是由于 money 不是索引的第一列，那么在查询中这个索引也不会被 MySQL 采用。

（5）如果 like 是以 `%` 开始，例如：
```
mysql> explain select * from company2 where name like '%3'\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: ALL
      possible_keys: NULL
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using where
    1 row in set(0.00 sec)
```
可见虽然在 name 上建有索引，但是由于 where 条件中 like 的值的“%”在第一位了，那么 MySQL 也不会采用这个索引。

（6）如果列类型是字符串，那么一定记得在 where 条件中把字符常量值用引号引起来，否则的话即便这个列上有索引，MySQL 也不会用到的，因为，MySQL 默认把输入的常量值进行转换以后才进行检索。如下面的例子中 company2 表中的 name 字段是字符型的，但是 SQL 语句中的条件值 294 是一个数值型值，因此即便在 name 上有索引，MySQL 也不能正确地用上索引，而是继续进行全表扫描。
```
mysql> explain select * from company2 where name = 294\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: ALL
      possible_keys: ind_company2_name
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using where
    1 row in set(0.00 sec)

mysql> explain select * from company2 where name = '294'\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: company2
               type: ref
      possible_keys: ind_company2_name
                key: ind_company2_name
            key_len: 23
                ref: const
               rows: 1
              Extra: Using where
    1 row in set(0.00 sec)
```
从上面的例子中可以看到，第一个 SQL 语句中把一个数值型常量赋值给了一个字符型的列 name，那么虽然在 name 列上有索引，但是也没有用到；而第二个 SQL 语句就可以正确使用索引。

## 查看索引使用情况
如果索引正在工作，`Handler_read_key` 的值将很高，这个值代表了一个行被索引值读的次数，很低的值表明增加索引得到的性能改善不高，因为索引并不经常使用。

`Handler_read_rnd_next` 的值高则意味着查询运行低效，并且应该建立索引补救。这个值的含义是在数据文件中读下一行的请求数。如果正进行大量的表扫描，`Handler_read_rnd_next` 的值较高，则通常说明表索引不正确或写入的查询没有利用索引，具体如下：
```
mysql> show status like 'Handler_read%';
+------------------------+----------+
|  Variable_name         |  Value   |
+------------------------+----------+
|  Handler_read_first    |  0       |
|  Handler_read_key      |  5       |
|  Handler_read_next     |  0       |
|  Handler_read_prev     |  0       |
|  Handler_read_rnd      |  0       |
|  Handler_read_rnd_next |  2055    |
+------------------------+----------+

6 rows in set(0.00 sec)
```
从上面的例子中可以看出，目前使用的 MySQL 数据库的索引情况并不理想。

> 本文大多摘自《深入浅出MySQL》。