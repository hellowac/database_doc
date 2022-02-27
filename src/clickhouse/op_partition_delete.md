# 删除指定分区

合理地设计分区键并利用分区的删除功能，就能够达到数据更新的目的。删除一个指定分区的语法如下所示：

```sql
ALTER TABLE tb_name DROP PARTITION partition_expr
```

假如现在需要更新partition_v2数据表整个7月份的数据，则可以先将7月份的分区删除：

```sql
ALTER TABLE partition_v2 DROP PARTITION 201907
```

然后将整个7月份的新数据重新写入，就可以达到更新的目的：

```sql
INSERT INTO partition_v2 VALUES ('A004-update','www.bruce.com', '2019-07-02'),…
```

查验数据表，可以看到7月份的数据已然更新：

```sql
SELECT * from partition_v2 ORDER BY EventTime
┌─ID───────────┬──────URL──────┬ EventTime  ┐
│ A001         │ www.nauu.com  │ 2019-05-02 │
│ A002         │ www.nauu1.com │ 2019-06-02 │
│ A004-update  │ www.bruce.com │ 2019-07-02 │
└──────────────┴───────────────┴────────────┘
```
