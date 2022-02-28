# 熔断机制

熔断是限制资源被过度使用的一种自我保护机制，当使用的资源数量达到阈值时，那么正在进行的操作会被自动中断。按照使用资源统计方式的不同，熔断机制可以分为两类。

## 1.根据时间周期的累积用量熔断

在这种方式下，系统资源的用量是按照时间周期累积统计的，当累积量达到阈值，则直到下个计算周期开始之前，该用户将无法继续进行操作。这种方式通过users.xml内的quotas标签来定义资源配额。以下面的配置为例：

```xml
<quotas>
  <default> <!-- 自定义名称 -->
    <interval>
      <duration>3600</duration><!-- 时间周期 单位：秒 -->
      <queries>0</queries>
      <errors>0</errors>
      <result_rows>0</result_rows>
      <read_rows>0</read_rows>
      <execution_time>0</execution_time>
    </interval>
  </default>
</quotas>
```

其中，各配置项的含义如下：

- **default**：表示自定义名称，全局唯一。
- **duration**：表示累积的时间周期，单位是秒。
- **queries**：表示在周期内允许执行的查询次数，0表示不限制。
- **errors**：表示在周期内允许发生异常的次数，0表示不限制。
- **result_row**：表示在周期内允许查询返回的结果行数，0表示不限制。
- **read_rows**：表示在周期内在分布式查询中，允许远端节点读取的数据行数，0表示不限制。
- **execution_time**：表示周期内允许执行的查询时间，单位是秒，0表示不限制。

由于上述示例中各配置项的值均为0，所以对资源配额不做任何限制。现在继续声明另外一组资源配额：

```xml
<limit_1>
  <interval>
    <duration>3600</duration>
    <queries>100</queries>
    <errors>100</errors>
    <result_rows>100</result_rows>
    <read_rows>2000</read_rows>
    <execution_time>3600</execution_time>
  </interval>
</limit_1>
```

为了便于演示，在这个名为limit_1的配额中，在1小时（3600秒）的周期内只允许100次查询。继续修改用户user_normal的配置，为它添加limit_1配额的引用：

```xml
<user_normal>
  <password></password>
  <networks>
    <ip>10.37.129.13</ip>
  </networks>
  <profile>normal</profile>
  <quota>limit_1</quota>
</user_normal>
```

最后使用user_normal用户登录，测试配额是否生效。在执行了若干查询以后，会发现之后的任何一次查询都将会得到如下异常：

```sql
Quota for user 'user_normal' for 1 hour has been exceeded. Total result rows: 149, max: 100. Interval will end at 2019-08-29 22:00:00. Name of quota template: 'limit_1'..
```

上述结果证明熔断机制已然生效。

## 2.根据单次查询的用量熔断

在这种方式下，系统资源的用量是按照单次查询统计的，而具体的熔断规则，则是由许多不同配置项组成的，这些配置项需要定义在用户profile中。如果某次查询使用的资源用量达到了阈值，则会被中断。以配置项max_memory_usage为例，它限定了单次查询可以使用的内存用量，在默认的情况下其规定不得超过10 GB，如果一次查询的内存用量超过10 GB，则会得到异常。需要注意的是，在单次查询的用量统计中，ClickHouse是以分区为最小单元进行统计的（不是数据行的粒度），这意味着单次查询的实际内存用量是有可能超过阈值的。

由于篇幅所限，完整的熔断配置请参阅官方手册，这里只列举个别的常用配置项。

首先介绍一组针对普通查询的熔断配置。

- （1）**max_memory_usage**：在单个ClickHouse服务进程中，运行一次查询限制使用的最大内存量，默认值为10 GB，其配置形式如下。

    ```xml
    <max_memory_usage>10000000000</max_memory_usage>
    ```

- （2）**max_memory_usage_for_user**：在单个ClickHouse服务进程中，以用户为单位进行统计，单个用户在运行查询时限制使用的最大内存量，默认值为0，即不做限制。

- （3）**max_memory_usage_for_all_queries**：在单个ClickHouse服务进程中，所有运行的查询累加在一起所限制使用的最大内存量，默认为0，即不做限制。

接下来介绍的是一组与数据写入和聚合查询相关的熔断配置。

- （1）**max_partitions_per_insert_block**：在单次INSERT写入的时候，限制创建的最大分区个数，默认值为100个。如果超出这个阈值，将会出现如下异常：

    ```xml
    Too many partitions for single INSERT block ……
    ```

- （2）**max_rows_to_group_by**：在执行GROUP BY聚合查询的时候，限制去重后聚合

    KEY的最大个数，默认值为0，即不做限制。当超过阈值时，其处理方式由group_by_overflow_mode参数决定。

- （3）**group_by_overflow_mode**：当max_rows_to_group_by熔断规则触发时，group_by_overflow_mode将会提供三种处理方式。

  - throw：抛出异常，此乃默认值。
  - break：立即停止查询，并返回当前数据。
  - any：仅根据当前已存在的聚合KEY继续完成聚合查询。

- （4）**max_bytes_before_external_group_by**：在执行GROUP BY聚合查询的时候，限制使用的最大内存量，默认值为0，即不做限制。当超过阈值时，聚合查询将会进一步借用本地磁盘。
