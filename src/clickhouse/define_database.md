# 数据库

数据库起到了命名空间的作用，可以有效规避命名冲突的问题，也为后续的数据隔离提供了支撑。

任何一张数据表，都必须归属在某个数据库之下。

创建数据库的完整语法如下所示：

```sql
CREATE DATABASE IF NOT EXISTS db_name [ENGINE = engine]
```

其中，`IF NOT EXISTS`表示如果已经存在一个同名的数据库，则会忽略后续的创建过程；`[ENGINE=engine]`表示数据库所使用的引擎类型.

数据库目前一共支持5种引擎，如下所示。

- **Ordinary**：默认引擎，在绝大多数情况下我们都会使用默认引擎，使用时无须刻意声明。在此数据库下可以使用任意类型的表引擎。
- **Dictionary**：字典引擎，此类数据库会自动为所有数据字典创建它们的数据表，关于数据字典的详细介绍会在第5章展开。
- **Memory**：内存引擎，用于存放临时数据。此类数据库下的数据表只会停留在内存中，不会涉及任何磁盘操作，当服务重启后数据会被清除。
- **Lazy**：日志引擎，此类数据库下只能使用Log系列的表引擎，关于Log表引擎的详细介绍会在第8章展开。
- **MySQL**：MySQL引擎，此类数据库下会自动拉取远端MySQL中的数据，并为它们创建MySQL表引擎的数据表。

在绝大多数情况下都只需使用默认的数据库引擎。例如执行下面的语句，即能够创建属于我们的第一个数据库：

```sql
CREATE DATABASE DB_TEST
```

默认数据库的实质是物理磁盘上的一个文件目录，所以在语句执行之后，ClickHouse便会在安装路径下创建DB_TEST数据库的文件目录：

```shell
# pwd
/chbase/data
# ls
DB_TEST  default  system
```

与此同时，在metadata路径下也会一同创建用于恢复数据库的DB_TEST.sql文件：

```shell
# pwd
/chbase/data/metadata
# ls
DB_TEST  DB_TEST.sql  default  system
```

使用SHOW DATABASES查询，即能够返回ClickHouse当前的数据库列表：

```shell
SHOW DATABASES
┌─name──────┐
│  DB_TEST  │
│  default  │
│  system   │
└───────────┘
```

使用USE查询可以实现在多个数据库之间进行切换，而通过SHOW TABLES查询可以查看当前数据库的数据表列表。删除一个数据库，则需要用到下面的DROP查询。

```sql
DROP DATABASE [IF EXISTS] db_name
```
