---
title: MySQL 中优化 SQL 语句的一般步骤
date: 2019-2-13 10:21:00
categories: [开发,数据库]
tags: [MySQL]
---

## 前言
当面对一个有 SQL 性能问题的数据库时，我们应该从何处入手来进行系统的分析，使得能够尽快定位问题 SQL 并尽快解决问题。

## show status
**通过 show status 命令了解各种 SQL 的执行频率。**

MySQL 客户端连接成功后，通过 `show[session|global] status` 命令可以提供服务器状态信息，也可以在操作系统上使用 `mysqladmin extended-status` 命令获得这些消息。`show[session|global] status` 可以根据需要加上参数“session”或者“global”来显示 session 级（当前连接）的统计结果和 global 级（自数据库上次启动至今）的统计结果。**如果不写，默认使用参数是“session”。**

下面的命令显示了当前 session 中所有统计参数的值：
```
mysql> show status like 'Com_%';
+------------------------------+---------+
|  Variable_name               |  Value  |
+------------------------------+---------+
|  Com_admin_commands          |  0      |
|  Com_alter_db                |  0      |
|  Com_alter_event             |  0      |
|  Com_alter_table             |  0      |
|  Com_analyze                 |  0      |
|  Com_backup_table            |  0      |
|  Com_begin                   |  0      |
|  Com_change_db               |  1      |
|  Com_change_master           |  0      |
|  Com_check                   |  0      |
|  Com_checksum                |  0      |
|  Com_commit                  |  0      |
......
```
Com_xxx 表示每个 xxx 语句执行的次数，我们通常比较关心的是以下几个统计参数。

- Com_select：执行 SELECT 操作的次数，一次查询只累加 1。
- Com_insert：执行 INSERT 操作的次数，对于批量插入的 INSERT 操作，只累加一次。
- Com_update：执行 UPDATE 操作的次数。
- Com_delete：执行 DELETE 操作的次数。

上面这些参数对于所有存储引擎的表操作都会进行累计。下面这几个参数只是针对 InnoDB 存储引擎的，累加的算法也略有不同。

- Innodb_rows_read：执行 SELECT 操作查询返回的行数。
- Innodb_rows_inserted：执行 INSERT 操作插入的行数。
- Innodb_rows_updated：执行 UPDATE 操作更新的行数。
- Innodb_rows_deleted：执行 DELETE 操作删除的行数。

通过以上几个参数，可以很容易地了解当前数据库的应用是以插入更新为主还是以查询操作为主，以及各种类型的 SQL 大致的执行比例是多少。对于更新操作的计数，是对执行次数的计数，不论提交还是回滚都会进行累加。

对于事务型的应用，通过 Com_commit 和 Com_rollback 可以了解事务提交和回滚的情况，对于回滚操作非常频繁的数据库，可能意味着应用编写存在问题。

此外，以下几个参数便于用户了解数据库的基本情况。

- Connections：试图连接 MySQL 服务器的次数。
- Uptime：服务器工作时间。
- Slow_queries：慢查询的次数。

## 定位 SQL 语句
**可以通过以下两种方式定位执行效率较低的 SQL 语句。**

- 通过慢查询日志定位那些执行效率较低的 SQL 语句，用 `--log-slow-queries[=file_name]` 选项启动时，mysqld 写一个包含所有执行时间超过 `long_query_time` 秒的 SQL 语句的日志文件。
- 慢查询日志在查询结束以后才记录，所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用 `show processlist` 命令查看当前 MySQL 在进行的线程，包括线程的状态、是否锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化。

## EXPLAIN
**通过 EXPLAIN 分析低效 SQL 的执行计划**

通过以上步骤查询到效率低的 SQL 语句后，可以通过 `EXPLAIN` 或者 `DESC` 命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序，比如想计算 2006 年所有公司的销售额，需要关联 sales 表和 company 表，并且对 moneys 字段做求和（sum）操作，相应 SQL 的执行计划如下：
```
mysql> explain select sum(moneys) from sales a,company b where a.company_id = b.id and a.year = 2006\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: a
               type: ALL
      possible_keys: NULL
                key: NULL
            key_len: NULL
                ref: NULL
               rows: 1000
              Extra: Using where
    **************************** 2. row *************************************
                 id: 1
        select_type: SIMPLE
              table: b
               type: ref
      possible_keys: ind_company_id
                key: ind_company_id
            key_len: 5
                ref: sakila.a.company_id
               rows: 1
              Extra: Using where; Using index
    2 rows in set (0.00 sec)
```
每个列的简单解释如下：

- select_type：表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（子查询中的第一个 SELECT）等。
- table：输出结果集的表。
- type：表示表的连接类型，性能由好到差的连接类型为 system（表中仅有一行，即常量表）、const（单表中最多有一个匹配行，例如 primary key 或者 unique index）、eq_ref（对于前面的每一行，在此表中只查询一条记录，简单来说，就是多表连接中使用 primary key 或者 unique index）、ref（与 eq_ref 类似，区别在于不是使用 primary key 或者 unique index，而是使用普通的索引）、ref_or_null（与 ref 类似，区别在于条件中包含对 NULL 的查询）、index_merge（索引合并优化）、unique_subquery（in 的后面是一个查询主键字段的子查询）、index_subquery（与 unique_subquery 类似，区别在于 in 的后面是查询非唯一索引字段的子查询）、range（单表中的范围查询）、index（对于前面的每一行，都通过查询索引来得到数据）、all（对于前面的每一行，都通过全表扫描来得到数据）。
- possible_keys：表示查询时，可能使用的索引。
- key：表示实际使用的索引。
- key_len：索引字段的长度。
- rows：扫描行的数量。
- Extra：执行情况的说明和描述。

## 确定问题并采取相应的优化措施
**经过以上步骤，基本就可以确认问题出现的原因。此时可以根据情况采取相应的措施，进行优化提高执行的效率。**

在上面的例子中，已经可以确认是对 a 表的全表扫描导致效率的不理想，那么对 a 表的 year 字段创建索引，具体如下：
```
mysql> create index ind_sales2_year on sales2(year);
Query OK,1000 rows affected(0.03 sec)
Records:1000 Duplicates: 0  Warnings: 0
```
创建索引后，再看一下这条语句的执行计划，具体如下：
```
mysql> explain select sum(moneys) from sales2 a,company2 b where a.company_id = b.id and a.year = 2006\G;
    **************************** 1. row *************************************
                 id: 1
        select_type: SIMPLE
              table: a
               type: ref
      possible_keys: ind_sales2_year
                key: ind_sales2_year
            key_len: 2
                ref: const
               rows: 1
              Extra: Using where
    **************************** 2. row *************************************
                 id: 1
        select_type: SIMPLE
              table: b
               type: ref
      possible_keys: ind_company_id
                key: ind_company_id
            key_len: 5
                ref: sakila.a.company_id
               rows: 1
              Extra: Using where; Using index
    2 rows in set (0.00 sec)
```
可以发现建立索引后对 a 表需要扫描的行数明显减少（从 1000 行减少到 1 行），可见索引的使用可以大大提高数据库的访问速度，尤其在表很庞大的时候这种优势更为明显。

> 本文大多摘自《深入浅出MySQL》。