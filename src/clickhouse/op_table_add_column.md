# 追加字段

假如需要对一张数据表追加新的字段，可以使用如下语法：

```sql
ALTER TABLE tb_name ADD COLUMN [IF NOT EXISTS] name [type] [default_expr] [AFTER name_after]
```

例如，在数据表的末尾增加新字段：

```sql
ALTER TABLE testcol_v1 ADD COLUMN OS String DEFAULT 'mac'
```

或是通过`AFTER`修饰符，在指定字段的后面增加新字段：

```sql
ALTER TABLE testcol_v1 ADD COLUMN IP String AFTER ID
```

对于数据表中已经存在的旧数据而言，新追加的字段会使用默认值补全。
