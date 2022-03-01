- [系统表](#系统表)
  - [metrics](#metrics)
  - [events](#events)
  - [asynchronous_metrics](#asynchronous_metrics)

# 系统表

参考官网: [System Tables](https://clickhouse.com/docs/en/operations/system-tables/)

在众多的SYSTEM系统表中，主要由以下三张表支撑了对ClickHouse运行指标的查询，它们分别是`metrics`、`events`和`asynchronous_metrics`。

## metrics

**metrics表用于统计ClickHouse服务在运行时，当前正在执行的高层次的概要信息，包括正在执行的查询总次数、正在发生的合并操作总次数等**。

该系统表的查询方法如下所示：

```sql
SELECT * FROM system.metrics LIMIT 5

┌─metric──────────┬─value──┬─description─────────────────────────────────────┐
│ Query           │     1  │ Number of executing queries                     │
│ Merge           │     0  │ Number of executing background merges           │
│ PartMutation    │     0  │ Number of mutations (ALTER DELETE/UPDATE)       │
│ ReplicatedFetch │     0  │ Number of data parts being fetched from replica │
│ ReplicatedSend  │     0  │ Number of data parts being sent to replicas     │
└─────────────────┴────────┴─────────────────────────────────────────────────┘
```

## events

**events用于统计ClickHouse服务在运行过程中已经执行过的高层次的累积概要信息，包括总的查询次数、总的SELECT查询次数等**.

该系统表的查询方法如下所示：

```sql
SELECT event, value FROM system.events LIMIT 5

┌─event───────────────────────────────────┬──value─┐
│ Query                                   │   165  │
│ SelectQuery                             │    92  │
│ InsertQuery                             │    14  │
│ FileOpen                                │  3525  │
│ ReadBufferFromFileDescriptorRead        │  6311  │
└─────────────────────────────────────────┴────────┘
```

## asynchronous_metrics

**asynchronous_metrics用于统计ClickHouse服务运行过程时，当前正在后台异步运行的高层次的概要信息，包括当前分配的内存、执行队列中的任务数量等。**

该系统表的查询方法如下所示：

```sql
SELECT * FROM system.asynchronous_metrics LIMIT 5
┌─metric───────────────────────────────────────┬─────value──────┐
│ jemalloc.background_thread.run_interval      │            0   │
│ jemalloc.background_thread.num_runs          │            0   │
│ jemalloc.background_thread.num_threads       │            0   │
│ jemalloc.retained                            │     79454208   │
│ jemalloc.mapped                              │    531341312   │
└──────────────────────────────────────────────┴────────────────┘
```
