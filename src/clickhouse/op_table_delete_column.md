# 删除已有字段

假如要删除某个字段，可以使用下面的语句：

```sql
ALTER TABLE tb_name DROP COLUMN [IF EXISTS] name
```

例如，执行下面的语句删除URL字段：

```sql
ALTER TABLE testcol_v1 DROP COLUMN URL
```

上述列字段在被删除之后，它的数据也会被连带删除。进一步来到testcol_v1的数据目录查验，会发现URL的数据文件已经被删除了：

```sql
# pwd
/chbase/data/data/default/testcol_v1/201907_2_2_0
# ll
total 56
-rw-r-----. 1 clickhouse clickhouse  28 Jul  2 21:02 EventTime.bin
-rw-r-----. 1 clickhouse clickhouse  30 Jul  2 21:02 ID.bin
-rw-r-----. 1 clickhouse clickhouse  30 Jul  2 21:02 IP.bin
-rw-r-----. 1 clickhouse clickhouse  30 Jul  2 21:02 OS.bin
省略…
```
