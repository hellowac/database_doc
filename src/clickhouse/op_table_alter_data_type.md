# 修改数据类型

如果需要改变表字段的数据类型或者默认值，需要使用下面的语法：

```sql
ALTER TABLE tb_name MODIFY COLUMN [IF EXISTS] name [type] [default_expr]
```

修改某个字段的数据类型，实质上会调用相应的toType转型方法。如果当前的类型与期望的类型不能兼容，则修改操作将会失败。

例如，将String类型的IP字段修改为IPv4类型是可行的：

```sql
ALTER TABLE testcol_v1 MODIFY COLUMN IP IPv4
```

而尝试将String类型转为UInt类型就会出现错误：

```sql
ALTER TABLE testcol_v1 MODIFY COLUMN OS UInt32
DB::Exception: Cannot parse string 'mac' as UInt32: syntax error at begin of string.
```
