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

### 数据库总大小

```sql
SELECT 
    sum(rows) AS `总行数`,
    formatReadableSize(sum(data_uncompressed_bytes)) AS `原始大小`,
    formatReadableSize(sum(data_compressed_bytes)) AS `压缩大小`,
    round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `压缩率`
FROM system.parts
```
