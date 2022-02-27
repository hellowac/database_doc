# 重置分区数据

如果数据表某一列的数据有误，需要将其重置为初始值，此时可以使用下面的语句实现：

```sql
ALTER TABLE tb_name CLEAR COLUMN column_name IN PARTITION partition_expr
```

对于默认值的含义，笔者遵循如下原则：如果声明了默认值表达式，则以表达式为准；否则以相应数据类型的默认值为准。

例如，执行下面的语句会重置partition_v2表内201908分区的URL数据重置。

```sql
ALTER TABLE partition_v2 CLEAR COLUMN URL in PARTITION 201908
```

查验数据后会发现，URL字段已成功被全部重置为空字符串了（String类型的默认值）。

```sql
SELECT * from partition_v2
┌───ID────┬─URL──┬───EventTime┐
│ A006-v1 │      │ 2019-08-05 │
│ A007-v1 │      │ 2019-08-20 │
└─────────┴──────┴────────────┘
```
