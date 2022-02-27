# 修改备注

做好信息备注是保持良好编程习惯的美德之一。追加备注的语法如下所示：

```sql
ALTER TABLE tb_name COMMENT COLUMN [IF EXISTS] name 'some comment'
```

例如，为ID字段增加备注：

```sql
ALTER TABLE testcol_v1 COMMENT COLUMN ID '主键ID'
```

使用DESC查询可以看到上述增加备注的操作已经生效：

```sql
DESC testcol_v1
┌─name────────┬─type───┬─comment─┐
│ ID          │ String │ 主键ID   │
└─────────────┴────────┴─────────┘
```
