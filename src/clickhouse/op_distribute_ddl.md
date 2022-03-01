# 分布式DDL执行

ClickHouse支持集群模式，一个集群拥有1到多个节点。

CREATE、ALTER、DROP、RENMAE及TRUNCATE这些DDL语句，都支持分布式执行。

这意味着，如果在集群中任意一个节点上执行DDL语句，那么集群中的每个节点都会以相同的顺序执行相同的语句。

这项特性意义非凡，它就如同批处理命令一样，省去了需要依次去单个节点执行DDL的烦恼。

将一条普通的DDL语句转换成分布式执行十分简单，只需加上`ON CLUSTER cluster_name`声明即可。

例如，执行下面的语句后将会对`ch_cluster`集群内的所有节点广播这条DDL语句：

```sql
CREATE TABLE partition_v3 ON CLUSTER ch_cluster(  -- 在 ch_cluster 集群上执行
  ID String,
  URL String,
  EventTime Date
) ENGINE =  MergeTree()
PARTITION BY toYYYYMM(EventTime)  -- 通过月份分区
ORDER BY ID
```

## 查询已有集群

```sql
:) SELECT cluster, host_name, host_address FROM system.clusters

Query id: fcd377f5-6a49-4ba5-aed8-e86f7cb96c0b

┌─cluster──────────────────────────────────────┬─host_name─────┬─host_address──┐
│ test_cluster_two_shards                      │ 127.0.0.1     │ 127.0.0.1     │
│ test_cluster_two_shards                      │ 127.0.0.2     │ 127.0.0.2     │
│ test_cluster_two_shards_internal_replication │ 127.0.0.1     │ 127.0.0.1     │
│ test_cluster_two_shards_internal_replication │ 127.0.0.2     │ 127.0.0.2     │
│ test_cluster_two_shards_localhost            │ localhost     │ ::1           │
│ test_cluster_two_shards_localhost            │ localhost     │ ::1           │
│ test_shard_localhost                         │ localhost     │ ::1           │
│ test_shard_localhost_secure                  │ localhost     │ ::1           │
│ test_unavailable_shard                       │ localhost     │ ::1           │
│ test_unavailable_shard                       │ localhost     │ ::1           │
└──────────────────────────────────────────────┴───────────────┴───────────────┘

14 rows in set. Elapsed: 0.002 sec.
```
