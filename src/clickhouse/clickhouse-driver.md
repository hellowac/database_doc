# Clickhouse-driver

原文: <https://github.com/mymarilyn/clickhouse-driver/blob/master/README.rst>

具有Native (TCP) 接口支持的 ClickHouse Python 驱动程序。

异步包装器可用在: <https://github.com/mymarilyn/aioch>

## 功能

* 基于外部数据的查询处理支持.
* Query查询设置.
* 压缩支持.
* TLS 支持.
* 类型支持:
  * Float32/64
  * [U]Int8/16/32/64/128/256
  * Date/Date32/DateTime('timezone')/DateTime64('timezone')
  * String/FixedString(N)
  * Enum8/16
  * Array(T)
  * Nullable(T)
  * Bool
  * UUID
  * Decimal
  * IPv4/IPv6
  * LowCardinality(T)
  * SimpleAggregateFunction(F, T)
  * Tuple(T1, T2, ...)
  * Nested
  * Map(key, value)
* Query查询进度信息。
* 基于Block块式的结果流传输.
* 读取查询配置文件信息.
* 接收服务器日志.
* 多主机支持.
* Python DB API 2.0 规范支持.
* 可选的 NumPy 数组支持.

## 文档

文档可在 <https://clickhouse-driver.readthedocs.io> 获得。

## 用法

与服务器通信有两种方式：

1. 使用纯客户端；
2. 使用数据库 API。

**纯客户端** 示例：

```python
>>> from clickhouse_driver import Client
>>>
>>> client = Client('localhost')
>>>
>>> client.execute('SHOW TABLES')
[('test',)]
>>> client.execute('DROP TABLE IF EXISTS test')
[]
>>> client.execute('CREATE TABLE test (x Int32) ENGINE = Memory')
[]
>>> client.execute(
...     'INSERT INTO test (x) VALUES',
...     [{'x': 100}]
... )
1
>>> client.execute('INSERT INTO test (x) VALUES', [[200]])
1
>>> client.execute(
...     'INSERT INTO test (x) '
...     'SELECT * FROM system.numbers LIMIT %(limit)s',
...     {'limit': 3}
... )
[]
>>> client.execute('SELECT sum(x) FROM test')
[(303,)]
```

**数据库API** 示例:

```python
>>> from clickhouse_driver import connect
>>>
>>> conn = connect('clickhouse://localhost')
>>> cursor = conn.cursor()
>>>
>>> cursor.execute('SHOW TABLES')
>>> cursor.fetchall()
[('test',)]
>>> cursor.execute('DROP TABLE IF EXISTS test')
>>> cursor.fetchall()
[]
>>> cursor.execute('CREATE TABLE test (x Int32) ENGINE = Memory')
>>> cursor.fetchall()
[]
>>> cursor.executemany(
...     'INSERT INTO test (x) VALUES',
...     [{'x': 100}]
... )
>>> cursor.rowcount
1
>>> cursor.executemany('INSERT INTO test (x) VALUES', [[200]])
>>> cursor.rowcount
1
>>> cursor.execute(
...     'INSERT INTO test (x) '
...     'SELECT * FROM system.numbers LIMIT %(limit)s',
...     {'limit': 3}
... )
>>> cursor.rowcount
0
>>> cursor.execute('SELECT sum(x) FROM test')
>>> cursor.fetchall()
[(303,)]
```
