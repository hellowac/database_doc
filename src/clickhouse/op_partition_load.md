# 卸载与装载分区

表分区可以通过`DETACH`语句卸载，分区被卸载后，它的物理数据并没有删除，而是被转移到了当前数据表目录的`detached`子目录下。而装载分区则是反向操作，它能够将`detached`子目录下的某个分区重新装载回去。卸载与装载这一对伴生的操作，常用于分区数据的迁移和备份场景。卸载某个分区的语法如下所示：

```sql
ALTER TABLE tb_name DETACH PARTITION partition_expr
```

例如，执行下面的语句能够将partition_v2表内整个8月份的分区卸载：

```sql
ALTER TABLE partition_v2 DETACH PARTITION 201908
```

此时再次查询这张表，会发现其中2019年8月份的数据已经没有了。而进入partition_v2的磁盘目录，则可以看到被卸载的分区目录已经被移动到了detached目录中：

```sql
# pwd
/chbase/data/data/default/partition_v2/detached
# ll
total 4
drwxr-x---. 2 clickhouse clickhouse 4096 Aug 31 23:16 201908_4_4_0
```

记住，一旦分区被移动到了detached子目录，就代表它已经脱离了ClickHouse的管理，ClickHouse并不会主动清理这些文件。这些分区文件会一直存在，除非我们主动删除或者使用ATTACH语句重新装载它们。装载某个分区的完整语法如下所示：

```sql
ALTER TABLE tb_name ATTACH PARTITION partition_expr
```

再次执行下面的语句，就可以将刚才已被卸载的201908分区重新装载回去：

```sql
ALTER TABLE partition_v2 ATTACH PARTITION 201908
```
