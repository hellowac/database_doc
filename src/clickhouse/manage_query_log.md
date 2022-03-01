# 查询日志

查询日志目前主要有6种类型，它们分别从不同角度记录了ClickHouse的操作行为。

所有查询日志在默认配置下都是关闭状态，需要在`config.xml`配置中进行更改，接下来分别介绍它们的开启方法。

在配置被开启之后，ClickHouse会为每种类型的查询日志自动生成相应的系统表以供查询。

## query_log

`query_log`是最常用的查询日志，它记录了ClickHouse服务中所有已经执行的查询记录，它的全局定义方式如下所示：

```xml
<query_log>
  <database>system</database>
  <table>query_log</table>
  <partition_by>toYYYYMM(event_date)</partition_by>
  <!--刷新周期-->
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</query_log>
```

如果只需要为某些用户单独开启query_log，也可以在user.xml的profile配置中按照下面的方式定义：

```xml
<log_queries> 1</log_queries>
```

query_log开启后，即可以通过相应的系统表对记录进行查询：

```sql
SELECT type,concat(substr(query,1,20),'...')query,read_rows, 
query_duration_ms AS duration FROM system.query_log LIMIT 6

┌─type─────────────────┬─query───────────────────┬─read_rows──┬─duration──┐
│ QueryStart           │ SELECT DISTINCT arra... │         0  │        0  │
│ QueryFinish          │ SELECT DISTINCT arra... │      2432  │       11  │
│ QueryStart           │ SHOW DATABASES...       │         0  │        0  │
│ QueryFinish          │ SHOW DATABASES...       │         3  │        1  │
│ ExceptionBeforeStart │ SELECT * FROM test_f... │         0  │        0  │
│ ExceptionBeforeStart │ SELECT * FROM test_f... │         0  │        0  │
└──────────────────────┴─────────────────────────┴────────────┴───────────┘
```

如上述查询结果所示，query_log日志记录的信息十分完善，涵盖了查询语句、执行时间、执行用户返回的数据量和执行用户等。

## query_thread_log

query_thread_log记录了所有线程的执行查询的信息，它的全局定义方式如下所示：

```xml
<query_thread_log>
  <database>system</database>
  <table>query_thread_log</table>
  <partition_by>toYYYYMM(event_date)</partition_by>
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</query_thread_log>
```

同样，如果只需要为某些用户单独开启该功能，可以在user.xml的profile配置中按照下面的方式定义：

```xml
<log_query_threads> 1</log_query_threads>
```

query_thread_log开启后，即可以通过相应的系统表对记录进行查询：

```sql
SELECT thread_name,concat(substr(query,1,20),'...')query,query_duration_ms AS duration,memory_usage AS memory FROM system.query_thread_log LIMIT 6

┌─thread_name─────┬─query───────────────────┬─duration───┬─memory──┐
│ ParalInputsProc │ SELECT DISTINCT arra... │        2   │ 210888  │
│ ParalInputsProc │ SELECT DISTINCT arra... │        3   │ 252648  │
│ AsyncBlockInput │ SELECT DISTINCT arra... │        3   │ 449544  │
│ TCPHandler      │ SELECT DISTINCT arra... │       11   │      0  │
│ TCPHandler      │ SHOW DATABASES...       │        2   │      0  │
└─────────────────┴─────────────────────────┴────────────┴─────────┘
```

如上述查询结果所示，query_thread_log日志记录的信息涵盖了线程名称、查询语句、执行时间和内存用量等。

## part_log

part_log日志记录了MergeTree系列表引擎的分区操作日志，其全局定义方式如下所示：

```xml
<part_log>
  <database>system</database>
  <table>part_log</table>
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</part_log>
```

part_log开启后，即可以通过相应的系统表对记录进行查询：

```sql
SELECT event_type AS type,table,partition_id,event_date FROM system.part_log

┌─type───────┬─table──────────────────────┬─partition_id──┬─event_date──┐
│ NewPart    │ summing_table_nested_v1    │ 201908        │ 2020-01-29  │
│ NewPart    │ summing_table_nested_v1    │ 201908        │ 2020-01-29  │
│ MergeParts │ summing_table_nested_v1    │ 201908        │ 2020-01-29  │
│ RemovePart │ ttl_table_v1               │ 201505        │ 2020-01-29  │
│ RemovePart │ summing_table_nested_v1    │ 201908        │ 2020-01-29  │
│ RemovePart │ summing_table_nested_v1    │ 201908        │ 2020-01-29  │
└────────────┴────────────────────────────┴───────────────┴─────────────┘
```

如上述查询结果所示，part_log日志记录的信息涵盖了操纵类型、表名称、分区信息和执行时间等。

## text_log

text_log日志记录了ClickHouse运行过程中产生的一系列打印日志，包括INFO、DEBUG和Trace，它的全局定义方式如下所示：

```xml
<text_log>
  <database>system</database>
  <table>text_log</table>
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</text_log>
```

text_log开启后，即可以通过相应的系统表对记录进行查询：

```sql
SELECT thread_name,
concat(substr(logger_name,1,20),'...')logger_name,
concat(substr(message,1,20),'...')message 
FROM system.text_log LIMIT 5

┌─thread_name────┬─logger_name─────────────┬─message────────────────────┐
│ SystemLogFlush │ SystemLog (system.me... │ Flushing system log...     │
│ SystemLogFlush │ SystemLog (system.te... │ Flushing system log...     │
│ SystemLogFlush │ SystemLog (system.te... │ Creating new table s...    │
│ SystemLogFlush │ system.text_log...      │ Loading data parts...      │
│ SystemLogFlush │ system.text_log...      │ Loaded data parts (0...    │
└────────────────┴─────────────────────────┴────────────────────────────┘
```

如上述查询结果所示，text_log日志记录的信息涵盖了线程名称、日志对象、日志信息和执行时间等。

## metric_log

metric_log日志用于将system.metrics和system.events中的数据汇聚到一起，它的全局定义方式如下所示：

```xml
<metric_log>
  <database>system</database>
  <table>metric_log</table>
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
  <collect_interval_milliseconds>1000</collect_interval_milliseconds>
</metric_log>
```

其中，`collect_interval_milliseconds`表示收集`metrics`和`events`数据的时间周期。`metric_log`开启后，即可以通过相应的系统表对记录进行查询。

除了上面介绍的系统表和查询日志外，ClickHouse还能够与众多的第三方监控系统集成，请参考相关资料。
