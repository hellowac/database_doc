# 嵌套

嵌套类型，顾名思义是一种嵌套表结构。

一张数据表，可以定义任意多个嵌套类型字段，但每个字段的嵌套层级只支持一级，即嵌套表内不能继续使用嵌套类型。

对于简单场景的层级关系或关联关系，使用嵌套类型也是一种不错的选择。

例如，下面的nested_test是一张模拟的员工表，它的所属部门字段就使用了嵌套类型：

```sql
CREATE TABLE nested_test (
  name String,
  age  UInt8 ,
  dept Nested(
  id UInt8,
  name String
)
) ENGINE = Memory;
```

ClickHouse的嵌套类型和传统的嵌套类型不相同，导致在初次接触它的时候会让人十分困惑。以上面这张表为例，如果按照它的字面意思来理解，会很容易理解成nested_test与dept是一对一的包含关系，其实这是错误的。不信可以执行下面的语句，看看会是什么结果：

```sql
INSERT INTO nested_test VALUES ('nauu',18, 10000, '研发部');
Exception on client:
Code: 53. DB::Exception: Type mismatch in IN or VALUES section. Expected: Array(UInt8). Got: UInt64
```

注意上面的异常信息，它提示期望写入的是一个Array数组类型。

**嵌套类型本质是一种多维数组的结构。嵌套表中的每个字段都是一个数组，并且行与行之间数组的长度无须对齐。**

所以需要把刚才的INSERT语句调整成下面的形式：

```sql
INSERT INTO nested_test VALUES ('bruce' , 30 , [10000,10001,10002], ['研发部','技术支持中心','测试部']);
--行与行之间,数组长度无须对齐
INSERT INTO nested_test VALUES ('bruce' , 30 , [10000,10001], ['研发部','技术支持中心']);
```

需要注意的是，在同一行数据内每个数组字段的长度必须相等。例如，在下面的示例中，由于行内数组字段的长度没有对齐，所以会抛出异常：

```sql
INSERT INTO nested_test VALUES ('bruce' , 30 , [10000,10001], ['研发部','技术支持中心', '测试部']); 
DB::Exception: Elements 'dept.id' and 'dept.name' of Nested data structure 'dept' (Array columns) have different array sizes..
```

在访问嵌套类型的数据时需要使用点符号，例如：

```sql
SELECT name, dept.id, dept.name FROM nested_test

┌─name──┬─dept.id────┬────dept.name───────────────────┐
│ bruce │ [16,17,18] │ ['研发部','技术支持中心','测试部'] │
└───────┴────────────┴────────────────────────────────┘
```
