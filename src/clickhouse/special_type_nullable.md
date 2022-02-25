# Nullable

原文：<https://clickhouse.com/docs/zh/sql-reference/data-types/nullable/>

准确来说，Nullable并不能算是一种独立的数据类型，它更像是一种辅助的修饰符，需要与基础数据类型一起搭配使用。

Nullable类型与Java8的Optional对象有些相似，它表示某个基础数据类型可以是Null值。

```sql
CREATE TABLE Null_TEST (
  c1 String,
  c2 Nullable(UInt8)
) ENGINE = TinyLog;

- 通过Nullable修饰后c2字段可以被写入Null值：

INSERT INTO Null_TEST VALUES ('nauu',null)
INSERT INTO Null_TEST VALUES ('bruce',20)

SELECT c1 , c2 ,toTypeName(c2) FROM Null_TEST

┌───c1───┬────c2───┬──toTypeName(c2)─┐
│ nauu   │ NULL    │ Nullable(UInt8) │
│ bruce  │ 20      │ Nullable(UInt8) │
└────────┴─────────┴─────────────────┘
```

**注意**：

- 它只能和基础类型搭配使用，不能用于数组和元组这些复合类型，也不能作为索引字段

- 慎用Nullable类型，包括Nullable的数据表，不然会使查询和写入性能变慢。【因为在正常情况下，每个列字段的数据会被存储在对应的[Column].bin文件中。如果一个列字段被Nullable类型修饰后，会额外生成一个[Column].null.bin文件专门保存它的Null值。这意味着在读取和写入数据时，需要一倍的额外文件操作。】
