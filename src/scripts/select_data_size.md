# 查询数据大小

## MYSQL

```sql
select
 table_schema as '数据库',
 table_name as '表名',
 table_rows as '记录数',
 truncate(data_length / 1024 / 1024, 2) as '数据容量(MB)',
 truncate(index_length / 1024 / 1024, 2) as '索引容量(MB)'
from
 information_schema.tables
order by
 data_length desc,
 index_length desc;
```

## ClickHouse

### 查看数据库容量、行数、压缩率

```sql
SELECT 
    sum(rows) AS `总行数`,
    formatReadableSize(sum(data_uncompressed_bytes)) AS `原始大小`,
    formatReadableSize(sum(data_compressed_bytes)) AS `压缩大小`,
    round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `压缩率`
FROM system.parts
```

输出:

```shell
┌────总行数─┬─原始大小──┬─压缩大小─┬─压缩率─┐
│ 326819026 │ 77.15 GiB │ 5.75 GiB │      7 │
└───────────┴───────────┴──────────┴────────┘

1 rows in set. Elapsed: 0.047 sec. Processed 1.04 thousand rows, 520.93 KB (21.95 thousand
rows/s., 11.02 MB/s.) 
```

### 查看数据表容量、行数、压缩率

```sql
SELECT 
    table AS `表名`,
    sum(rows) AS `总行数`,
    formatReadableSize(sum(data_uncompressed_bytes)) AS `原始大小`,
    formatReadableSize(sum(data_compressed_bytes)) AS `压缩大小`,
    round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `压缩率`
FROM system.parts
WHERE table IN ('temp_1')
GROUP BY table
```

输出:

```shell
┌─表名───┬──总行数─┬─原始大小───┬─压缩大小──┬─压缩率─┐
│ temp_1 │ 3127523 │ 838.21 MiB │ 60.04 MiB │      7 │
└────────┴─────────┴────────────┴───────────┴────────┘

1 rows in set. Elapsed: 0.008 sec.
```

### 查看数据表分区信息

```sql
--查看测试表在19年12月的分区信息
SELECT 
    partition AS `分区`,
    sum(rows) AS `总行数`,
    formatReadableSize(sum(data_uncompressed_bytes)) AS `原始大小`,
    formatReadableSize(sum(data_compressed_bytes)) AS `压缩大小`,
    round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `压缩率`
FROM system.parts
WHERE (database IN ('default')) AND (table IN ('temp_1')) AND (partition LIKE '2019-12-%')
GROUP BY partition
ORDER BY partition ASC
```

输出:

```shell
┌─分区───────┬─总行数─┬─原始大小──┬─压缩大小───┬─压缩率─┐
│ 2019-12-01 │     24 │ 6.17 KiB  │ 2.51 KiB   │     41 │
│ 2019-12-02 │   9215 │ 2.45 MiB  │ 209.74 KiB │      8 │
│ 2019-12-03 │  17265 │ 4.46 MiB  │ 453.78 KiB │     10 │
│ 2019-12-04 │  27741 │ 7.34 MiB  │ 677.25 KiB │      9 │
│ 2019-12-05 │  31500 │ 8.98 MiB  │ 469.30 KiB │      5 │
│ 2019-12-06 │    157 │ 37.50 KiB │ 4.95 KiB   │     13 │
│ 2019-12-07 │    110 │ 32.75 KiB │ 3.86 KiB   │     12 │
└────────────┴────────┴───────────┴────────────┴────────┘

7 rows in set. Elapsed: 0.005 sec. 
```

### 查看数据表字段的信息

```sql
SELECT 
    column AS `字段名`,
    any(type) AS `类型`,
    formatReadableSize(sum(column_data_uncompressed_bytes)) AS `原始大小`,
    formatReadableSize(sum(column_data_compressed_bytes)) AS `压缩大小`,
    sum(rows) AS `行数`
FROM system.parts_columns
WHERE (database = 'default') AND (table = 'temp_1')
GROUP BY column
ORDER BY column ASC
```

输出:

```shell
┌─字段名───────────┬─类型─────┬─原始大小───┬─压缩大小───┬────行数─┐
│ a                │ String   │ 23.83 MiB  │ 134.13 KiB │ 3127523 │
│ b                │ String   │ 19.02 MiB  │ 127.72 KiB │ 3127523 │
│ c                │ String   │ 5.97 MiB   │ 49.09 KiB  │ 3127523 │
│ d             │ String   │ 3.95 MiB   │ 532.86 KiB │ 3127523 │
│ e                │ String   │ 5.17 MiB   │ 49.47 KiB  │ 3127523 │
│ totalDate        │ DateTime │ 11.93 MiB  │ 1.26 MiB   │ 3127523 │
└──────────────────┴──────────┴────────────┴────────────┴─────────┘
```
