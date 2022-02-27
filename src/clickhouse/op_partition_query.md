# 查询分区信息

`ClickHouse`内置了许多`system系统表`，用于查询自身的状态信息。

其中`parts系统表`专门用于查询数据表的分区信息。

例如执行下面的语句，就能够得到数据表`partition_v2`的分区状况：

```sql
SELECT partition_id,name,table,database FROM system.parts WHERE table = 'partition_v2'
┌─partition_id───┬─name───────────┬────table─────┬─database┐
│ 201905         │ 201905_1_1_0_6 │ partition_v2 │ default │
│ 201910         │ 201910_3_3_0_6 │ partition_v2 │ default │
│ 201911         │ 201911_4_4_0_6 │ partition_v2 │ default │
│ 201912         │ 201912_5_5_0_6 │ partition_v2 │ default │
└────────────────┴────────────────┴──────────────┴─────────┘
```

如上所示，目前`partition_v2`共拥有4个分区，其中`partition_id`或者`name`等同于分区的主键，可以基于它们的取值确定一个具体的分区。
