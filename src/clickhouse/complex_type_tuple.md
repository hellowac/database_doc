# 元组

元组类型由`1～n`个元素组成，每个元素之间允许设置不同的数据类型，且彼此之间不要求兼容。

元组同样支持类型推断，其推断依据仍然以最小存储代价为原则。

与数组类似，元组也可以使用两种方式定义:

- **常规方式**`tuple(T)`：

    ```sql

    SELECT tuple(1,'a',now()) AS x, toTypeName(x)
    ┌─x─────────────────────────────┬─toTypeName(tuple(1, 'a', now()))──┐
    │ (1,'a','2019-08-28 21:36:32') │ Tuple(UInt8, String, DateTime)    │
    └───────────────────────────────┴───────────────────────────────────┘
    ```

- **简写方式（T）**:

```sql
SELECT (1,2.0,null) AS x, toTypeName(x)
┌─x────────────┬─toTypeName(tuple(1, 2., NULL))───────────┐
│ (1,2,NULL)   │ Tuple(UInt8, Float64, Nullable(Nothing)) │
└──────────────┴──────────────────────────────────────────┘
```

在定义表字段时，元组需要指定明确的元素类型：

```sql
CREATE TABLE Tuple_TEST (
  c1 Tuple(String,Int8)
) ENGINE = Memory;
```

元素类型和泛型的作用类似，可以进一步保障数据质量。在数据写入的过程中会进行类型检查。

例如，写入`INSERT INTO Tuple_TEST VALUES(('abc',123))`是可行的，而写入`INSERT INTO Tuple_TEST VALUES(('abc','efg'))`则会报错。
