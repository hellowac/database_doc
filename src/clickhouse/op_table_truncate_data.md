# 清空数据表

假设需要将表内的数据全部清空，而不是直接删除这张表，则可以使用`TRUNCATE`语句，它的完整语法如下所示：

```sql
TRUNCATE TABLE [IF EXISTS] [db_name.]tb_name
```

例如执行下面的语句，就能将`db_test.testcol_v2`的数据一次性清空：

```sql
TRUNCATE TABLE db_test.testcol_v2
```
