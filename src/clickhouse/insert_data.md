- [数据的写入](#数据的写入)
  - [使用VALUES格式](#使用values格式)
  - [使用指定格式](#使用指定格式)
  - [使用SELECT子句形式](#使用select子句形式)

# 数据的写入

参考原文: [INSERT INTO 语句](https://clickhouse.com/docs/zh/sql-reference/statements/insert-into/)

INSERT语句支持三种语法范式，三种范式各有不同，可以根据写入的需求灵活运用。

## 使用VALUES格式

语法:

```sql
INSERT INTO [db.]table [(c1, c2, c3…)] VALUES (v11, v12, v13…), (v21, v22, v23…), ...
```

其中，c1、c2、c3是列字段声明，可省略。

VALUES后紧跟的是由元组组成的待写入数据，通过下标位与列字段声明一一对应。

数据支持批量声明写入，多行数据之间使用逗号分隔。

例如执行下面的语句，将批量写入多条数据：

```sql
INSERT INTO partition_v2 VALUES 
('A0011','www.nauu.com', '2019-10-01'), 
('A0012','www.nauu.com', '2019-11-20'),
('A0013','www.nauu.com', '2019-12-20')
```

在使用VALUES格式的语法写入数据时，支持加入表达式或函数，例如：

```sql
INSERT INTO partition_v2 VALUES ('A0014',toString(1+2), now())
```

## 使用指定格式

语法:

```sql
INSERT INTO [db.]table [(c1, c2, c3…)] FORMAT format_name data_set
```

ClickHouse支持多种数据格式（更多格式可参见官方手册），以常用的CSV格式写入为例：

```sql
INSERT INTO partition_v2 FORMAT CSV \
'A0017','www.nauu.com', '2019-10-01' \
'A0018','www.nauu.com', '2019-10-01'
```

## 使用SELECT子句形式

语法：

```sql
INSERT INTO [db.]table [(c1, c2, c3…)] SELECT ...
```

通过SELECT子句可将查询结果写入数据表，假设需要将partition_v1的数据写入partition_v2，则可以使用下面的语句：

```sql
INSERT INTO partition_v2 SELECT * FROM partition_v1
```

在通过SELECT子句写入数据的时候，同样也支持加入表达式或函数，例如：

```sql
INSERT INTO partition_v2 SELECT 'A0020', 'www.jack.com', now()
```

虽然VALUES和SELECT子句的形式都支持声明表达式或函数，但是表达式和函数会带来额外的性能开销，从而导致写入性能的下降。

所以如果追求极致的写入性能，就应该尽可能避免使用它们。

ClickHouse内部所有的数据操作都是面向Block数据块的，所以INSERT查询最终会将数据转换为Block数据块。也正因如此，INSERT语句在单个数据块的写入过程中是具有原子性的。

在默认的情况下，每个数据块最多可以写入`1048576`行数据（由`max_insert_block_size`参数控制）。也就是说，如果一条`INSERT语句`写入的数据少于`max_insert_block_size`行，那么这批数据的写入是具有原子性的，即要么全部成功，要么全部失败。

> **注意**:
>
> 只有在ClickHouse服务端处理数据的时候才具有这种原子写入的特性。
>
> 例如使用JDBC或者HTTP接口时。因为`max_insert_block_size`参数在使用`CLI命令行`或者`INSERT SELECT`子句写入时是不生效的。
