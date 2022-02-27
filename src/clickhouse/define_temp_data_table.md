# 临时表

ClickHouse也有临时表的概念，创建临时表的方法是在普通表的基础之上添加TEMPORARY关键字.

语法：

```sql
CREATE TEMPORARY TABLE [IF NOT EXISTS] table_name (
    name1 [type] [DEFAULT|MATERIALIZED|ALIAS expr],
    name2 [type] [DEFAULT|MATERIALIZED|ALIAS expr],
)
```

两点特殊之处:

- 它的生命周期是会话绑定的，所以它只支持Memory表引擎，如果会话结束，数据表就会被销毁；
- 临时表不属于任何数据库，所以在它的建表语句中，既没有数据库参数也没有表引擎参数。

> 如果临时表和普通表名称相同，会出现什么状况呢？
>
> 临时表的优先级是大于普通表的。当两张数据表名称相同的时候，会优先读取临时表的数据。

在ClickHouse的日常使用中，通常不会刻意使用临时表。它更多被运用在ClickHouse的内部，是数据在集群间传播的载体。
