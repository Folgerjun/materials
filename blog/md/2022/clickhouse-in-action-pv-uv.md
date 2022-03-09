---
title: ClickHouse 实战整理 - 统计 pv uv
date: 2022-03-08 21:10:55
categories: [开发,数据库]
tags: [ClickHouse]
---


> [ClickHouse 官方文档](https://clickhouse.com/docs/en/)
>
> 文中举例均为简单示例，根据业务自行扩展。

实际业务场景中很多会需要统计 pv uv 值，比如页面的 pv uv 值，商品、店铺的 pv uv 值……

这边举一个简单的例子，比如说统计商品访问的 pv uv：

- 简单的源数据表创建如下：

```sql
CREATE TABLE IF NOT EXISTS test.product_operation
(
    ts_date_time  DateTime COMMENT '事件触发时间',
    gen_date_time DateTime COMMENT '入库时间',
    product_id    UInt64 COMMENT '商品ID',
    user_id       UInt64 COMMENT '用户ID',
    operation     UInt16 COMMENT '商品操作类型（1：访问，2：购买，3：收藏）'
) ENGINE = MergeTree()
      ORDER BY (toDate(ts_date_time), operation);
```

像这种数据量可能很大，又需要预先聚合计算以减小获取耗时的，我们添加 [物化视图](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/aggregatingmergetree/#ju-he-wu-hua-shi-tu-de-shi-li) 来帮助我们做这个事。

- 创建按天聚合数据的物化视图：

```sql
CREATE MATERIALIZED VIEW IF NOT EXISTS test.product_operation_day_pv_uv_mv
            ENGINE = AggregatingMergeTree()
                ORDER BY (ts_date, product_id, operation)
AS
SELECT toDate(ts_date_time)        as ts_date,
       product_id,
       operation,
       sumState(1)                 as pv,
       uniqState(user_id, ts_date) as uv
FROM test.product_operation
GROUP BY ts_date, product_id, operation;
```

- 创建总的聚合数据的物化视图：

```sql
CREATE MATERIALIZED VIEW IF NOT EXISTS test.product_operation_total_pv_uv_mv
            ENGINE = AggregatingMergeTree()
                ORDER BY (product_id, operation)
AS
SELECT product_id,
       operation,
       sumState(1)        as pv,
       uniqState(user_id) as uv
FROM test.product_operation
GROUP BY product_id, operation;
```

- 现在我们往源数据表中插入一条数据：

```sql
INSERT INTO test.product_operation(ts_date_time, gen_date_time, product_id, user_id, operation)
VALUES (toDateTime('2022-01-26 10:10:10'), now(), 1, 1, 1)
```

- 再查询：

> 忽略时区问题导致的时间显示

```sql
SELECT * FROM  test.product_operation ORDER BY gen_date_time DESC;
```

| ts\_date\_time      | gen\_date\_time     | product\_id | user\_id | operation |
| :------------------ | :------------------ | :---------- | :------- | :-------- |
| 2022-01-26 02:10:10 | 2022-01-26 03:54:36 | 1           | 1        | 1         |

- 查询视图（day）：

```sql
SELECT ts_date, product_id, operation, sumMerge(pv) AS pv, uniqMerge(uv) AS uv
FROM test.product_operation_day_pv_uv_mv GROUP BY ts_date, product_id, operation;
```

| ts\_date   | product\_id | operation | pv   | uv   |
| :--------- | :---------- | :-------- | :--- | :--- |
| 2022-01-26 | 1           | 1         | 1    | 1    |

- 查询视图（total）：

```sql
SELECT product_id, operation, sumMerge(pv) AS pv, uniqMerge(uv) AS uv
FROM test.product_operation_total_pv_uv_mv GROUP BY product_id, operation;
```

| product\_id | operation | pv   | uv   |
| :---------- | :-------- | :--- | :--- |
| 1           | 1         | 1    | 1    |

- 再往源数据插入数据：

```sql
INSERT INTO test.product_operation(ts_date_time, gen_date_time, product_id, user_id, operation)
VALUES (toDateTime('2022-01-26 23:10:10'), now(), 1, 1, 1);
```

- 查询视图：

| ts\_date   | product\_id | operation | pv   | uv   |
| :--------- | :---------- | :-------- | :--- | :--- |
| 2022-01-26 | 1           | 1         | 2    | 1    |

| product\_id | operation | pv   | uv   |
| :---------- | :-------- | :--- | :--- |
| 1           | 1         | 2    | 1    |

- 换个 `user_id` 插入数据：

```sql
INSERT INTO test.product_operation(ts_date_time, gen_date_time, product_id, user_id, operation)
VALUES (toDateTime('2022-01-26 23:20:10'), now(), 1, 2, 1);
```

- 查询视图：

| ts\_date   | product\_id | operation | pv   | uv   |
| :--------- | :---------- | :-------- | :--- | :--- |
| 2022-01-25 | 1           | 1         | 3    | 2    |

| product\_id | operation | pv   | uv   |
| :---------- | :-------- | :--- | :--- |
| 1           | 1         | 3    | 2    |

- 再换个 `ts_date_time` 插入数据：

```sql
INSERT INTO test.product_operation(ts_date_time, gen_date_time, product_id, user_id, operation)
VALUES (toDateTime('2022-01-27 10:20:10'), now(), 1, 1, 1),
       (toDateTime('2022-01-27 11:20:10'), now(), 1, 2, 1);
```

- 查询视图：

| ts\_date   | product\_id | operation | pv   | uv   |
| :--------- | :---------- | :-------- | :--- | :--- |
| 2022-01-26 | 1           | 1         | 3    | 2    |
| 2022-01-27 | 1           | 1         | 2    | 2    |

| product\_id | operation | pv   | uv   |
| :---------- | :-------- | :--- | :--- |
| 1           | 1         | 5    | 2    |

---

综上，实现不难，clickhouse 丰富的聚合函数可以节省很多操作。

还可以用 [Kafka引擎](https://clickhouse.com/docs/zh/engines/table-engines/integrations/kafka/) 做数据的增量同步，结合视图。