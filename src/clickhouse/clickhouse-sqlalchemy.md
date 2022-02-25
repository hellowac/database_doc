- [Clickhouse-SQLAlchemy](#clickhouse-sqlalchemy)
  - [安装](#安装)
    - [接口支持](#接口支持)
  - [连接参数](#连接参数)
    - [驱动选项](#驱动选项)
  - [功能](#功能)
    - [SQLAlchemy 声明式支持](#sqlalchemy-声明式支持)
    - [基本的DDL(数据定义)支持](#基本的ddl数据定义支持)
    - [基本 INSERT 子句支持](#基本-insert-子句支持)
    - [常见的 SQLAlchemy 链式查询方法](#常见的-sqlalchemy-链式查询方法)
    - [高级 INSERT 子句支持](#高级-insert-子句支持)
    - [用于查询处理的外部数据](#用于查询处理的外部数据)
    - [支持的 ClickHouse 特别的 SQL](#支持的-clickhouse-特别的-sql)
  - [覆盖默认查询设置](#覆盖默认查询设置)
  - [运行测试](#运行测试)

# Clickhouse-SQLAlchemy

原文: <https://github.com/xzkostyan/clickhouse-sqlalchemy/blob/master/README.rst>

[SQLAlchemy]: http://docs.sqlalchemy.org/en/latest/
[clickhouse-driver]: https://github.com/mymarilyn/clickhouse-driver
[requests]: https://2.python-requests.org/en/latest/
[requests.Session]: https://requests.readthedocs.io/en/master/user/advanced/#session-objects

**ClickHouse SQLAlchemy** [SQLAlchemy]的连接Clickhouse数据库的方言版本。

## 安装

可以使用 pip 安装该软件包：

```text
`pip install clickhouse-sqlalchemy`
```

### 接口支持

- native 方式: 通过 [clickhouse-driver](https://github.com/mymarilyn/clickhouse-driver) 【推荐】【TCP】
- http 方式: 通过 [requests]

## 连接参数

ClickHouse SQLAlchemy 使用下面的语法作为连接字符串:

```text
'clickhouse+<driver>://<user>:<password>@<host>:<port>/<database>[?key=value..]'
```

注意:

- *driver* 是驱动选项. 可选: `http`, `native`. 默认为 `http`.
- *database* 是要连接的数据库. 默认为 `default`数据库.

### 驱动选项

可以在查询字符串中指定几个选项。

**HTTP**

- `port` 是ClickHouse绑定的端口. 默认为 `8123`.
- `timeout` 以秒为单位的超时. 无默认值.
- `protocol` 要使用的协议. 可选: `http`, `https`. 默认为 `http`.

安装ClickHouse后，默认的测试连接字符串是：

```text
'clickhouse://default:@localhost/test'
```

当您使用 nginx 作为 ClickHouse 的代理服务器时，服务器连接字符串可能如下所示：

```text
'clickhouse://user:password@example.com:8124/test?protocol=https'
```

这里的`8124`是代理端口。

如果您需要控制底层 HTTP 连接，请将 [requests.Session] 实例传递给 `create_engine()`，如下所示：

```python
from sqlalchemy import create_engine
from requests import Session

uri = 'clickhouse://default:@localhost/test'

engine = create_engine(uri, connect_args={'http_session': Session()})
```

**Native**

请注意，`native`连接方式未加密。 包括用户/密码在内的所有数据都以纯文本形式传输。 通过不受信任的网络进行通信时，您应该通过 `SSH` 或 `VPN`（例如）使用此连接。

安装ClickHouse后，默认的测试连接字符串是：

```text
'clickhouse+native://default:@localhost/test'
```

所有连接字符串参数都代理到 clickhouse-driver。 参考其[parameters](https://clickhouse-driver.readthedocs.io/en/latest/api.html#clickhouse_driver.connection.Connection)

## 功能

### SQLAlchemy 声明式支持

ORM声明式和构造表的函数式两种方式都支持：

```python
from sqlalchemy import create_engine, Column, MetaData, literal

from clickhouse_sqlalchemy import Table, make_session, get_declarative_base, types, engines

uri = 'clickhouse://default:@localhost/test'

engine = create_engine(uri)
session = make_session(engine)
metadata = MetaData(bind=engine)

Base = get_declarative_base(metadata=metadata)

class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)
    other_value = Column(
        types.DateTime,
        clickhouse_codec=('DoubleDelta', 'ZSTD'),
    )

    __table_args__ = (
        engines.Memory(),
    )

another_table = Table('another_rate', metadata,
    Column('day', types.Date, primary_key=True),
    Column('value', types.Int32, server_default=literal(1)),
    engines.Memory()
)
```

以声明方式创建的表名具有小写字母，单词由下划线命名约定分隔。 但是您可以通过 SQLAlchemy 的 `__tablename__` 属性轻松设置您自己的表名。

### 基本的DDL(数据定义)支持

您可以发出简单的 DDL。 例如 CREATE/DROP 表：

```python
table = Rate.__table__
table.create()
another_table.create()


another_table.drop()
table.drop()
```

### 基本 INSERT 子句支持

简单的批量插入：

```python
from datetime import date, timedelta
from sqlalchemy import func

today = date.today()
rates = [{'day': today - timedelta(i), 'value': 200 - i} for i in range(100)]

# Emits single INSERT statement.
session.execute(table.insert(), rates)
```

### 常见的 SQLAlchemy 链式查询方法

`order_by`, `filter`, `limit`, `offset`等都是支持的：

```python
session.query(func.count(Rate.day)) \
    .filter(Rate.day > today - timedelta(20)) \
    .scalar()

session.query(Rate.value) \
    .order_by(Rate.day.desc()) \
    .first()

session.query(Rate.value) \
    .order_by(Rate.day) \
    .limit(10) \
    .all()

session.query(func.sum(Rate.value)) \
    .scalar()
```

### 高级 INSERT 子句支持

INSERT FROM SELECT 语句:

```python
from sqlalchemy import cast

# Labels must be present.
select_query = session.query(
    Rate.day.label('day'),
    cast(Rate.value * 1.5, types.Int32).label('value')
).subquery()

# Emits single INSERT FROM SELECT statement
session.execute(
    another_table.insert()
    .from_select(['day', 'value'], select_query)
)
```

许多但不是所有的 SQLAlchemy 特性都支持开箱即用。

UNION ALL 示例：

```python
from sqlalchemy import union_all

select_rate = session.query(
    Rate.day.label('date'),
    Rate.value.label('x')
)
select_another_rate = session.query(
    another_table.c.day.label('date'),
    another_table.c.value.label('x')
)

union_all(select_rate, select_another_rate).execute().fetchone()
```

### 用于查询处理的外部数据

目前可以与native方式的接口一起使用:

```python
ext = Table(
    'ext', metadata, Column('x', types.Int32),
    clickhouse_data=[(101, ), (103, ), (105, )], extend_existing=True
)

rv = session.query(Rate) \
    .filter(Rate.value.in_(session.query(ext.c.x))) \
    .execution_options(external_tables=[ext]) \
    .all()

print(rv)
```

### 支持的 ClickHouse 特别的 SQL

- **SELECT query**:
  - **WITH TOTALS**
  - **SAMPLE**
  - **lambda functions: x -> expr**
  - **JOIN**

示例请参考 [tests](https://github.com/xzkostyan/clickhouse-sqlalchemy/tree/master/tests)

## 覆盖默认查询设置

设置较低的查询优先级并限制执行请求的最大线程数。

```python
rv = session.query(func.sum(Rate.value)) \
    .execution_options(settings={'max_threads': 2, 'priority': 10}) \
    .scalar()

print(rv)
```

## 运行测试

```shell
mkvirtualenv testenv && python setup.py test
```

`pip` 将自动安装所有必需的模块进行测试。
