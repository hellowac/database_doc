# 移动数据表

原文参考: [RENAME语法](https://clickhouse.com/docs/zh/sql-reference/statements/rename/)

在Linux系统中，`mv`命令的本意是将一个文件从原始位置A移动到目标位置B，但是如果位置A与位置B相同，则可以变相实现重命名的作用。

ClickHouse的`RENAME`查询就与之有着异曲同工之妙，`RENAME语句`的完整语法如下所示：

```sql
RENAME TABLE [db_name11.]tb_name11 TO [db_name12.]tb_name12, [db_name21.]tb_name21 TO [db_name22.]tb_name22, ...
```

RENAME可以修改数据表的名称，如果将原始数据库与目标数据库设为不同的名称，那么就可以实现数据表在两个数据库之间移动的效果。

例如在下面的例子中，`testcol_v1`从`default`默认数据库被移动到了`db_test`数据库，同时数据表被重命名为`testcol_v2`：

```sql
RENAME TABLE default.testcol_v1 TO db_test.testcol_v2
```

需要注意的是，数据表的移动只能在单个节点的范围内。

换言之，数据表移动的目标数据库和原始数据库必须处在同一个服务节点内，而不能是集群中的远程节点。
