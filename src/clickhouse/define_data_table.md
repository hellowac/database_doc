# 数据表

ClickHouse目前提供了三种最基本的建表方法。

**常规定义方法**

> 语法：
>
> ```sql
> CREATE TABLE [IF NOT EXISTS] [db_name.]table_name (
>     name1 [type] [DEFAULT|MATERIALIZED|ALIAS expr],
>     name2 [type] [DEFAULT|MATERIALIZED|ALIAS expr],
>     省略…
> ) ENGINE = engine
> ```
>
> 使用[db_name.]参数可以为数据表指定数据库，如果不指定此参数，则默认会使用default数据库:
>
> ```sql
> CREATE TABLE hits_v1 ( 
>   Title String,
>   URL String ,
>   EventTime DateTime
> ) ENGINE = Memory;
> ```
>
> 末尾的`ENGINE`参数，它被用于指定数据表的引擎。**`表引擎`决定了数据表的特性，也决定了数据将会被如何存储及加载。**
>
> 示例中使用的Memory表引擎，是ClickHouse最简单的表引擎，数据只会被保存在内存中，在服务重启时数据会丢失。

**复制其他表的结构**

> 语法：
>
> ```sql
> CREATE TABLE [IF NOT EXISTS] [db_name1.]table_name AS [db_name2.] table_name2 [ENGINE = engine]
> ```
>
> 这种方式支持在不同的数据库之间复制表结构
>
> ```sql
> --创建新的数据库
> CREATE DATABASE IF NOT EXISTS new_db 
> --将default.hits_v1的结构复制到new_db.hits_v1
> CREATE TABLE IF NOT EXISTS new_db.hits_v1 AS default.hits_v1 ENGINE = TinyLog
> ```
>
> 上述语句将会把`default.hits_v1`的表结构原样复制到`new_db.hits_v1`，并且ENGINE表引擎可以与原表不同

**通过SELECT子句创建**

> 语法：
>
> ```sql
> CREATE TABLE [IF NOT EXISTS] [db_name.]table_name ENGINE = engine AS SELECT …
> ```
>
> 在这种方式下，不仅会根据SELECT子句建立相应的表结构，同时还会将SELECT子句查询的数据顺带写入, 例如：
>
> ```sql
> CREATE TABLE IF NOT EXISTS hits_v1_1 ENGINE = Memory AS SELECT * FROM hits_v1
> ```

**DESC查询**

> ClickHouse和大多数数据库一样，使用DESC查询可以返回数据表的定义结构。
>
> ```sql
> :) desc hits_v1
> 
> DESCRIBE TABLE  hits_v1
> 
> Query id: eaa69aa2-5cd9-4ff8-9b70-7f0960aa33d8
> 
> ┌─name──────┬─type─────┬─default_type─┬─default_expression─┬─comment─┬─codec_expression─┬─ttl_expression─┐
> │ Title     │ String   │              │                    │         │                  │                │
> │ URL       │ String   │              │                    │         │                  │                │
> │ EventTime │ DateTime │              │                    │         │                  │                │
> └───────────┴──────────┴──────────────┴────────────────────┴─────────┴──────────────────┴────────────────┘
> 
> 3 rows in set. Elapsed: 0.001 sec.
> ```

**DROP删除表**

> 如果想删除一张数据表，则可以使用下面的DROP语句：
>
> ```sql
> DROP TABLE [IF EXISTS] [db_name.]table_name
> ```
>
> ```sql
> :) DROP　table default.hits_v1
> 
> DROP TABLE default.hits_v1
> 
> Query id: 2deeda6b-9922-41c9-b2af-5e8e3efbddfe
> 
> Ok.
> 
> 0 rows in set. Elapsed: 0.002 sec.
> ```
