# 分区表

数据分区（partition）和数据分片（shard）是完全不同的两个概念。

数据分区是针对本地数据而言的，是数据的一种纵向切分; 而数据分片是数据的一种横向切分。

数据分区对于一款OLAP数据库而言意义非凡：借助数据分区，在后续的查询过程中能够跳过不必要的数据目录，从而提升查询的性能。

合理地利用分区特性，还可以变相实现数据的更新操作，因为数据分区支持删除、替换和重置操作。

假设数据表按照月份分区，那么数据就可以按月份的粒度被替换更新。

分区虽好，但不是所有的表引擎都可以使用这项特性，目前只有合并树（`MergeTree`）家族系列的表引擎才支持数据分区。

创建表时，由`PARTITION BY`指定分区键，例如使用了日期字段作为分区键，并将其格式化为年月的形式:

```sql
CREATE TABLE partition_v1 ( 
  ID String,
  URL String,
  EventTime Date
) ENGINE =  MergeTree()
PARTITION BY toYYYYMM(EventTime) 
ORDER BY ID
```

接着写入不同月份的测试数据：

```sql
INSERT INTO partition_v1 VALUES 
('A000','www.nauu.com', '2019-05-01'),
('A001','www.brunce.com', '2019-06-02')
```

最后通过`system.parts`系统表，查询数据表的分区状态：

```sql
SELECT table,partition,path from system.parts WHERE table = 'partition_v1' 
┌─table────────┬──partition─┬─path───────────────────────────────────────────┐
│ partition_v1 │ 201905     │ /chbase/data/default/partition_v1/201905_1_1_0/│
│ partition_v1 │ 201906     │ /chbase/data/default/partition_v1/201906_2_2_0/│
└──────────────┴────────────┴────────────────────────────────────────────────┘
```

> partition_v1按年月划分后，目前拥有两个数据分区，且每个分区都对应一个独立的文件目录，用于保存各自部分的数据。

**合理设计分区键非常重要，通常会按照数据表的查询场景进行针对性设计。**

例如：

```sql
SELECT * FROM  partition_v1 WHERE EventTime ='2019-05-01'
```

那么在后续的查询过程中，可以利用分区索引跳过6月份的分区目录，只加载5月份的数据，从而带来查询的性能提升。

当然，使用不合理的分区键也会适得其反，分区键不应该使用粒度过细的数据字段。例如，按照小时分区，将会带来分区数量的急剧增长，从而导致性能下降。
