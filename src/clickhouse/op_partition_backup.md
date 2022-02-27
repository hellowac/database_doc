- [备份与还原分区](#备份与还原分区)
  - [导出文件备份](#导出文件备份)
  - [通过快照表备份](#通过快照表备份)
  - [按分区备份](#按分区备份)
    - [使用FREEZE备份](#使用freeze备份)
    - [使用FETCH备份](#使用fetch备份)

# 备份与还原分区

ClickHouse自身提供了多种备份数据的方法，根据数据规模的不同，可以选择不同的形式。

## 导出文件备份

如果数据的体量较小，可以通过dump的形式将数据导出为本地文件。例如执行下面的语句将test_backup的数据导出：

```shell
> clickhouse-client --query="SELECT * FROM test_backup" > /chbase/test_backup.tsv
```

将备份数据再次`导恢复数据`，则可以执行下面的语句：

```sql
> cat /chbase/test_backup.tsv | clickhouse-client --query "INSERT INTO test_backup FORMAT TSV"
```

上述这种dump形式的优势在于，可以利用SELECT查询并筛选数据，然后按需备份。如果是备份整个表的数据，也可以直接复制它的整个目录文件，例如：

```sql
> mkdir -p /chbase/backup/default/ & cp -r /chbase/data/default/test_backup /chbase/backup/default/
```

## 通过快照表备份

快照表实质上就是普通的数据表，它通常按照业务规定的备份频率创建，例如按天或者按周创建。

所以首先需要建立一张与原表结构相同的数据表，然后再使用INSERT INTO SELECT句式，点对点地将数据从原表写入备份表。

假设数据表`test_backup`需要按日进行备份，现在为它创建当天的备份表：

```sql
CREATE TABLE test_backup_0206 AS test_backup
```

有了备份表之后，就可以点对点地备份数据了，例如：

```sql
INSERT INTO TABLE test_backup_0206 SELECT * FROM test_backup
```

如果考虑到容灾问题，也可以将备份表放置在不同的ClickHouse节点上，此时需要将上述SQL语句改成远程查询的形式：

```sql
INSERT INTO TABLE test_backup_0206 SELECT * FROM remote('ch5.nauu.com:9000', 'default', 'test_backup', 'default')
```

## 按分区备份

基于数据分区的备份，ClickHouse目前提供了`FREEZE`与`FETCH`两种方式，现在分别介绍它们的使用方法。

### 使用FREEZE备份

FREEZE的完整语法如下所示：

参考原文: <https://clickhouse.com/docs/zh/sql-reference/statements/alter/#alter_freeze-partition>

```sql
ALTER TABLE tb_name FREEZE PARTITION partition_expr
```

分区在被备份之后，会被统一保存到ClickHouse根路径/shadow/N子目录下。

其中，N是一个自增长的整数，它的含义是备份的次数（FREEZE执行过多少次），具体次数由shadow子目录下的increment.txt文件记录。

而分区备份实质上是对原始目录文件进行硬链接操作，所以并不会导致额外的存储空间。整个备份的目录会一直向上追溯至data根路径的整个链路：

```shell
/data/[database]/[table]/[partition_folder]
```

例如执行下面的语句，会对数据表partition_v2的201908分区进行备份：

```sql
:) ALTER TABLE partition_v2 FREEZE PARTITION 201908
```

进入shadow子目录，即能够看到刚才备份的分区目录：

```shell
# pwd
/chbase/data/shadow/1/data/default/partition_v2
# ll
total 4
drwxr-x---. 2 clickhouse clickhouse 4096 Sep  1 00:22 201908_5_5_0
```

**对于备份分区的还原操作，则需要借助ATTACH装载分区的方式来实现。**

**这意味着如果要还原数据，首先需要主动将shadow子目录下的分区文件复制到相应数据表的detached目录下，然后再使用ATTACH语句装载。**

### 使用FETCH备份

FETCH只支持ReplicatedMergeTree系列的表引擎，它的完整语法如下所示：

```sql
ALTER TABLE tb_name FETCH PARTITION partition_id FROM zk_path
```

其工作原理与`ReplicatedMergeTree`同步数据的原理类似，FETCH通过指定的`zk_path`找到`ReplicatedMergeTree`的所有副本实例，然后从中选择一个最合适的副本，并下载相应的分区数据。例如执行下面的语句：

```sql
ALTER TABLE test_fetch FETCH PARTITION 2019 FROM '/clickhouse/tables/01/test_fetch'
```

表示指定将test_fetch的2019分区下载到本地，并保存到对应数据表的detached目录下，目录如下所示：

```sql
data/default/test_fetch/detached/2019_0_0_0
```

与FREEZE一样，对于备份分区的还原操作，也需要借助ATTACH装载分区来实现。

FREEZE和FETCH虽然都能实现对分区文件的备份，但是它们并不会备份数据表的元数据。

所以说如果想做到万无一失的备份，还需要对数据表的元数据进行备份，它们是`/data/metadata`目录下的`[table].sql`文件。目前这些元数据需要用户通过复制的形式单独备份。
