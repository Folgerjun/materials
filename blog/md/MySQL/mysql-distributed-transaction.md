---
title: MySQL 中分布式事务的使用
date: 2019-2-12 10:35:26
categories: [开发,数据库]
tags: [MySQL]
---

## 前言
MySQL 从 5.0.3 开始支持分布式事务，当前分布式事务只支持 InnoDB 存储引擎。一个分布式事务会涉及多个行动，这些行动本身是事务性的。所有行动都必须一起成功完成，或者一起被回滚。

## 分布式事务的原理
**在 MySQL 中，使用分布式事务的应用程序涉及一个或多个资源管理器和一个事务管理器。**

- 资源管理器（RM）用于提供通向事务资源的途经。**数据库服务器是一种资源管理器。**该管理器必须可以提交或回滚由 RM 管理的事务。例如，多台 MySQL 数据库作为多台资源管理器或者几台 MySQL 服务器和几台 Oracle 服务器作为资源管理器。
- 事务管理器（TM）用于协调作为一个分布式事务一部分的事务。**TM 与管理每个事务的 RMs 进行通讯。**一个分布式事务中各个单个事务均是分布式事务的“分支事务”。分布式事务和各分支通过一种命名方法进行标识。

MySQL 执行 XA MySQL 时，MySQL 服务器相当于一个用于管理分布式事务中的 XA 事务的资源管理器。与 MySQL 服务器连接的客户端相当于事务管理器。

**要执行一个分布式事务，必须知道这个分布式事务涉及到了哪些资源管理器，并且把每个资源管理器的事务执行到事务可以被提交或回滚时。**根据每个资源管理器报告的有关执行情况的内容，这些分支事务必须作为一个原子性操作全部提交或回滚。要管理一个分布式事务，必须要考虑任何组件或连接网络可能会故障。

用于执行分布式事务的过程使用两阶段提交，发生时间在由分布式事务的各个分支需要进行的行动已经被执行之后。

- 在第一阶段，所有的分支被预备好。即它们被 TM 告知要准备提交。通常，这意味着用于管理分支的每个 RM 会记录对于被稳定保存的分支的行动。分支指示是否它们可以这么做。这些结果被用于第二阶段。
- 在第二阶段，TM 告知 RMs 是否要提交或回滚。如果在预备分支时，所有的分支指示它们将能够提交，则所有的分支被告知要提交。如果在预备时，有任何分支指示它将不能提交，则所有分支被告知回滚。

**在有些情况下，一个分布式事务可能会使用一阶段提交。例如，当一个事务管理器发现，一个分布式事务只由一个事务资源组成（即单一分支），则该资源可以被告知同时进行预备和提交。**

## 分布式事务的语法
分布式事务（XA 事务）的 SQL 语法主要包括：

`XA {START|BEGIN} xid [JOIN|RESUME]`

XA START xid 用于启动一个带给定 xid 值的 XA 事务。每个 XA 事务必须有一个唯一的 xid 值，因此该值当前不能被其他的 XA 事务使用。

xid 是一个 XA 事务标识符，用来唯一标识一个分布式事务。xid 值由客户端提供，或由 MySQL 服务器生成。xid 值包含 1~3 个部分：

`xid: gtrid [, bqual [, formatID ]]`

- gtrid 是一个分布式事务标识符，相同的分布式事务应该使用相同的 gtrid，这样可以明确知道 xa 事务属于哪个分布式事务。
- bqual 是一个分支限定符，默认值是空串。对于一个分布式事务中的每个分支事务，bqual 值必须是唯一的。
- formatID 是一个数字，用于标识由 gtrid 和 bqual 值使用的格式，默认值是 1。

下面其他 XA 语法中用到的 xid 值，都必须和 START 操作使用的 xid 值相同，也就是表示对这个启动的 XA 事务进行操作。
```
XA END xid [SUSPEND [FOR MIGRATE]]
XA PREPARE xid
```
使事务进入 PREPARE 状态，也就是两阶段提交的第一个提交阶段。
```
XA COMMIT xid [ONE PHASE]
XA ROLLBACK xid
```
这两个命令用来提交或者回滚具体的分支事务。也就是两阶段提交的第二个提交阶段，分支事务被实际的提交或者回滚。
```
XA RECOVER  返回当前数据库中处于 PREPARE 状态的分支事务的详细信息。
```

**分布式的关键在于如何确保分布式事务的完整性，以及在某个分支出现问题时的故障解决。**XA 的相关命令就是提供给应用如何在多个独立的数据库之间进行分布式事务的管理，包括启动一个分支事务、使事务进入准备阶段以及事务的实际提交回滚操作等，如下所示的例子演示了一个简单的分布式事务的执行，事务的内容是在 DB1 中插入一条记录，同时在 DB2 中更新一条记录，两个操作作为同一事务提交或者回滚。

**↓↓分布式事务例子↓↓**

|session_1 in DB1|session_2 in DB2|
|:-:|:-:|
|在数据库 DB1 中启动一个分布式事务的一个分支事务，xid 的 gtrid 为“test”，bqual 为“db1”：|在数据库 DB2 中启动分布式事务“test”的另外一个分支事务，xid 的 gtrid 为“test”，bqual 为“db2”：|
|mysql> xa start 'test','db1';|mysql> xa start 'test','db2';|
|Query OK,0 rows affected(0.00 sec)| Query OK,0 rows affected(0.00 sec)|
|分支事务 1 在表 actor 中插入一条记录：|分支事务 2 在表 film_actor 中更新了 23 条记录：|
|mysql> insert into actor (actor_id, first_name, last_name) values (301, 'Simon', 'Tom');|mysql> update film_actor set last_update = now() where actor_id = 178;|
|Query OK,1 row affected(0.00 sec)|Query OK,23 rows affected(0.04 sec) Rows matched:23 Changed:23 Warnings:0|
|对分支事务 1 进行第一阶段提交，进入 prepare 状态：|对分支事务 2 进行第一阶段提交，进入 prepare 状态：|
|mysql> xa end 'test','db1';|mysql> xa end 'test','db2';|
|Query OK,0 rows affected(0.00 sec)|Query OK,0 rows affected(0.00 sec)|
|mysql> xa prepare 'test','db1';|mysql> xa prepare 'test','db2';|
|Query OK,0 rows affected(0.02 sec)|Query OK,0 rows affected(0.02 sec)|
|||
|用 xa recover 命令查看当前分支事务状态：|用 xa recover 命令查看当前分支事务状态：|
|mysql> xa recover \G|mysql> xa recover \G|
|formatID: 1|formatID: 1|
|gtrid_length: 4|gtrid_length: 4|
|bqual_length: 3|bqual_length: 3|
|data: testdb1|data: testdb2|
|1 row in set(0.00 sec)|1 row in set(0.00 sec)|
|两个事务都进入准备提交阶段，如果之前遇到任何错误，都应该回滚所有的分支，以确保分布式事务的正确。|两个事务都进入准备提交阶段，如果之前遇到任何错误，都应该回滚所有的分支，以确保分布式事务的正确。|
|提交分支事务 1：|提交分支事务 2：|
|mysql> xa commit 'test','db1';|mysql> xa commit 'test','db2';|
|Query OK,0 rows affected(0.03 sec)|Query OK,0 rows affected(0.03 sec)|
|两个事务都到达准备提交阶段后，一旦开始进行提交操作，就需要确保全部的分支都提交成功。|两个事务都到达准备提交阶段后，一旦开始进行提交操作，就需要确保全部的分支都提交成功。|

## 存在的问题
虽然 MySQL 支持分布式事务，但是在测试过程中，还是发现存在一些问题。

如果分支事务在达到 prepare 状态时，数据库异常重新启动，服务器重新启动以后，可以继续对分支事务进行提交或者回滚的操作，但是提交的事务没有写 binlog，存在一定的隐患，可能导致使用 binlog 恢复丢失部分数据。如果存在复制的数据库，则有可能导致主从数据库的数据不一致。

如果分支事务的客户端连接异常中止，那么数据库会自动回滚未完成的分支事务，如果此时分支事务已经执行到 prepare 状态，那么这个分布式事务的其他分支可能已经成功提交，如果这个分支回滚，可能导致分布式事务的不完整，丢失部分分支事务的内容。

如果分支事务在执行到 prepare 状态时，数据库异常，且不能再正常启动，需要使用备份和 binlog 来恢复数据，那么那些在 prepare 状态的分支事务因为并没有记录到 binlog，所以不能通过 binlog 进行恢复，在数据库恢复后，将丢失这部分的数据。

## 总结
MySQL 的分布式事务还存在比较严重的缺陷，在数据库或者应用异常的情况下，可能会导致分布式事务的不完整。如果应用对于数据的完整性要求不是很高，则可以考虑使用。如果应用对事务的完整性有比较高的要求，则不太推荐使用分布式事务。

> 以上大多摘自《深入浅出MySQL》