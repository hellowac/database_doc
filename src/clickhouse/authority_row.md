# 数据行级权限

数据权限是整个权限体系中的第三层防护，它决定了一个用户能够看到什么数据。数据权限使用databases标签定义，它是用户定义中的一项选填设置。database通过定义用户级别的查询过滤器来实现数据的行级粒度权限，它的定义规则如下所示：

```xml
<databases>
  <database_name><!--数据库名称-->
    <table_name><!--表名称-->
      <filter> id < 10</filter><!--数据过滤条件-->
        </table_name>
      </database_name>
```

其中，database_name表示数据库名称；table_name表示表名称；而filter则是权限过滤的关键所在，它等同于定义了一条WHERE条件子句，与WHERE子句类似，它支持组合条件。现在用一个示例说明。这里还是用user_normal，为它追加databases定义：

```xml
<user_normal>
  ……
  <databases>
    <default><!--默认数据库-->
      <test_row_level><!—表名称-->
        <filter>id < 10</filter>
          </test_row_level>

        <!—支持组合条件 
        <test_query_all>
          <filter>id <= 100 or repo >= 100</filter>
            </test_query_all> -->
          </default>
        </databases>
```

基于上述配置，通过为user_normal用户增加全局过滤条件，实现了该用户在数据表default.test_row_level的行级粒度数据上的权限设置。test_row_level的表结构如下所示：

```sql
CREATE TABLE test_row_level(
  id UInt64,
  name UInt64
)
ENGINE = MergeTree()
ORDER BY id
```

下面验证权限是否生效。首先写入测试数据：

```sql
INSERT INTO TABLE test_row_level VALUES (1,100),(5,200),(20,200),(30,110)
```

写入之后，登录user_normal用户并查询这张数据表：

```sql
SELECT * FROM test_row_level

┌─id──┬─name──┐
│  1  │  100  │
│  5  │  200  │
└─────┴───────┘
```

可以看到，在返回的结果数据中，只包含`<filter> id<10 </filter>`的部分，证明权限设置生效了。

那么数据权限的设定是如何实现的呢？进一步分析它的执行日志：

```sql
Expression
    Expression
        Filter –增加了过滤的步骤
            MergeTreeThread
```

可以发现，上述代码在普通查询计划的基础之上自动附加了Filter过滤的步骤。

对于数据权限的使用有一点需要明确，在使用了这项功能之后，PREWHERE优化将不再生效，例如执行下面的查询语句：

```sql
SELECT * FROM test_row_level where name = 5
```

此时如果使用了数据权限，那么这条SQL将不会进行PREWHERE优化；反之，如果没有设置数据权限，则会进行PREWHERE优化，例如：

```sql
InterpreterSelectQuery: MergeTreeWhereOptimizer: condition "name = 5" moved to PREWHERE
```

所以，是直接利用ClickHouse的内置过滤器，还是通过拼接WHERE查询条件的方式实现行级数据权限，需要用户在具体的使用场景中进行权衡。