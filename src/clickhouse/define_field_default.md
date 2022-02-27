# 字段默认值表达式

表字段支持三种默认值表达式的定义方法，分别是`DEFAULT`、`MATERIALIZED`和`ALIAS`。

无论使用哪种形式，表字段一旦被定义了默认值，它便不再强制要求定义数据类型，因为ClickHouse会根据默认值进行类型推断。

如果同时对表字段定义了数据类型和默认值表达式，则以明确定义的数据类型为主，例如:

```sql
CREATE TABLE dfv_v1 ( 
  id String,
  c1 DEFAULT 1000,  -- c1字段没有定义数据类型，默认值为整型1000；
  c2 String DEFAULT c1  -- c2字段定义了数据类型和默认值，且默认值等于c1, 由于同时定义了数据类型和默认值，所以它最终的数据类型来自明确定义的String。
) ENGINE = TinyLog
```

现在写入测试数据：

```sql
INSERT INTO dfv_v1(id) VALUES ('A000')
```

在写入之后执行以下查询：

```sql
:) SELECT c1, c2, toTypeName(c1), toTypeName(c2) from dfv_v1

Query id: 2508b947-02c8-4676-baca-658d90af2765

┌───c1─┬─c2───┬─toTypeName(c1)─┬─toTypeName(c2)─┐
│ 1000 │ 1000 │ UInt16         │ String         │
└──────┴──────┴────────────────┴────────────────┘

1 rows in set. Elapsed: 0.002 sec.
```

默认值表达式的三种定义方法之间也存在着不同之处:

- **数据写入**：在数据写入时，只有`DEFAULT类型`的字段可以出现在`INSERT语句`中。而`MATERIALIZED`和`ALIAS`都不能被显式赋值，它们只能依靠计算取值。例如试图为`MATERIALIZED类型`的字段写入数据，将会得到如下的错误。

    ```sql
    DB::Exception: Cannot insert column URL, because it is MATERIALIZED column..
    ```

- **数据查询**：在数据查询时，只有`DEFAULT类型`的字段可以通过`SELECT *`返回。而`MATERIALIZED`和`ALIAS`类型的字段不会出现在`SELECT*`查询的返回结果集中。

- **数据存储**：在数据存储时，只有`DEFAULT`和`MATERIALIZED类型`的字段才支持持久化。如果使用的表引擎支持物理存储（例如TinyLog表引擎），那么这些列字段将会拥有物理存储。而`ALIAS类型`的字段不支持持久化，它的取值总是需要依靠计算产生，数据不会落到磁盘。

**使用ALTER语句修改默认值**

```sql
ALTER TABLE [db_name.]table MODIFY COLUMN col_name DEFAULT value
```

**修改动作并不会影响数据表内先前已经存在的数据。但是默认值的修改有诸多限制**，例如在合并树表引擎中，它的主键字段是无法被修改的；而某些表引擎则完全不支持修改（例如TinyLog）。
