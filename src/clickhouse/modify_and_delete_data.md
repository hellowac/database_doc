# 数据的删除与修改

ClickHouse提供了`DELETE`和`UPDATE`的能力，这类操作被称为`Mutation`查询，它可以看作`ALTER`语句的变种。

虽然`Mutation`能最终实现修改和删除，但不能完全以通常意义上的`UPDATE`和`DELETE`来理解，我们必须清醒地认识到它的不同：

- 首先，`Mutation语句`是一种 **“很重”** 的操作，更适用于批量数据的修改和删除；

- 其次，它不支持事务，一旦语句被提交执行，就会立刻对现有数据产生影响，无法回滚；
  
- 最后，`Mutation语句`的执行是一个异步的后台过程，语句被提交之后就会立即返回。
  
- 所以这并不代表具体逻辑已经执行完毕，它的具体执行进度需要通过`system.mutations`系统表查询。

## 数据删除

DELETE语句的完整语法如下所示:

```sql
ALTER TABLE [db_name.]table_name DELETE WHERE filter_expr
```

数据删除的范围由WHERE查询子句决定。例如，执行下面语句可以删除partition_v2表内所有ID等于A003的数据：

```sql
ALTER TABLE partition_v2 DELETE WHERE ID = 'A003'
```

由于演示的数据很少，DELETE操作给人的感觉和常用的OLTP数据库无异。但是我们心中应该要明白这是一个异步的后台执行动作。

再次进入数据目录，让我们看看删除操作是如何实现的：

```shell
# pwd
/chbase/data/data/default/partition_v2
# ll
total 52
drwxr-x---. 2 clickhouse clickhouse 4096 Jul  6 15:03 201905_1_1_0
drwxr-x---. 2 clickhouse clickhouse 4096 Jul  6 15:03 201905_1_1_0_6
省略…
drwxr-x---. 2 clickhouse clickhouse 4096 Jul  6 15:03 201909_5_5_0
drwxr-x---. 2 clickhouse clickhouse 4096 Jul  6 15:03 201909_5_5_0_6
drwxr-x---. 2 clickhouse clickhouse 4096 Jul  6 15:02 detached
-rw-r-----. 1 clickhouse clickhouse    1 Jul  6 15:02 format_version.txt
-rw-r-----. 1 clickhouse clickhouse   89 Jul  6 15:03 mutation_6.txt
```

可以发现，在执行了DELETE操作之后数据目录发生了一些变化。每一个原有的数据目录都额外增加了一个同名目录，并且在末尾处增加了_6的后缀。此外，目录下还多了一个名为mutation_6.txt的文件，mutation_6.txt文件的内容如下所示：

```shell
# cat mutation_6.txt
format version: 1
create time: 2019-07-06 15:03:27
commands: DELETE WHERE ID = \'A003\'
```

原来mutation_6.txt是一个日志文件，它完整地记录了这次DELETE操作的执行语句和时间，而文件名的后缀_6与新增目录的后缀对应。那么后缀的数字从何而来呢？继续查询system.mutations系统表，一探究竟：

```sql
SELECT database, table ,mutation_id, block_numbers.number as num ,is_done FROM system.mutations

┌─database──┬────table─────┬──mutation_id───┬───num──┬─is_done─┐
│ default   │ partition_v2 │ mutation_6.txt │ [6]    │ 1       │
└───────────┴──────────────┴────────────────┴────────┴─────────┘
```

至此，整个Mutation操作的逻辑就比较清晰了。

- 每执行一条ALTER DELETE语句，都会在mutations系统表中生成一条对应的执行计划，当is_done等于1时表示执行完毕。
- 与此同时，在数据表的根目录下，会以mutation_id为名生成与之对应的日志文件用于记录相关信息。
- 而数据删除的过程是以数据表的每个分区目录为单位，将所有目录重写为新的目录，新目录的命名规则是在原有名称上加上`system.mutations.block_numbers.number`。

数据在重写的过程中会将需要删除的数据去掉。旧的数据目录并不会立即删除，而是会被标记成非激活状态（active为0）。
等到MergeTree引擎的下一次合并动作触发时，这些非激活目录才会被真正从物理意义上删除。

## 数据修改

数据修改除了需要指定具体的列字段之外，整个逻辑与数据删除别无二致，它的完整语法如下所示：

```sql
ALTER TABLE [db_name.]table_name UPDATE column1 = expr1 [, ...] WHERE filter_expr
```

UPDATE支持在一条语句中同时定义多个修改字段，分区键和主键不能作为修改字段。例如，执行下面的语句即能够根据WHERE条件同时修改partition_v2内的URL和OS字段：

```sql
ALTER TABLE partition_v2 
UPDATE URL = 'www.wayne.com', OS = 'mac' 
WHERE ID IN (
    SELECT ID 
    FROM partition_v2 
    WHERE EventTime = '2019-06-01'
)
```
