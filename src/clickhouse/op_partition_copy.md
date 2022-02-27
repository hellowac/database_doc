# 复制分区数据

ClickHouse支持将A表的分区数据复制到B表，这项特性可以用于快速数据写入、多表间数据同步和备份等场景，它的完整语法如下：

```sql
ALTER TABLE B REPLACE PARTITION partition_expr FROM A
```

不过需要注意的是，并不是任意数据表之间都能够相互复制，它们还需要满足两个前提条件：

- 两张表需要拥有相同的分区键；
- 它们的表结构完全相同。

再执行下面的语句：

```sql
ALTER TABLE partition_v2 REPLACE PARTITION 201908 FROM partition_v1
```

即能够将partition_v1的整个201908分区中的数据复制到partition_v2：

```sql
SELECT * from partition_v2 ORDER BY EventTime
┌─────ID───────┬─────URL───────┬──EventTime─┐
│ A000         │ www.nauu.com  │ 2019-05-01 │
│ A001         │ www.nauu.com  │ 2019-05-02 │
省略…
│ A004-update  │ www.bruce.com │ 2019-07-02 │
│ A006-v1      │ www.v1.com    │ 2019-08-05 │
│ A007-v1      │ www.v1.com    │ 2019-08-20 │
└──────────────┴───────────────┴────────────┘
```
