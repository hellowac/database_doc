- [时间类型](#时间类型)
  - [DateTime](#datetime)
  - [DateTime64](#datetime64)
  - [Date](#date)
  - [Date32](#date32)

# 时间类型

时间类型分为`DateTime`、`DateTime64`、`Date`、`Date32`四类。ClickHouse目前没有时间戳类型。

时间类型最高的精度是秒，也就是说，如果需要处理毫秒、微秒等大于秒分辨率的时间，则只能借助`UInt`类型实现.

## DateTime

DateTime类型包含时、分、秒信息，精确到秒，支持使用字符串形式写入：

```sql
CREATE TABLE Datetime_TEST (
    c1 Datetime
) ENGINE = Memory

--以字符串形式写入
INSERT INTO Datetime_TEST VALUES('2019-06-22 00:00:00')
 
 SELECT c1, toTypeName(c1) FROM Datetime_TEST

┌──────────c1─┬─toTypeName(c1)───────────┐
│ 2019-06-22 00:00:00 │  DateTime        │
└─────────────────────┴──────────────────┘
```

## DateTime64

DateTime64可以记录亚秒，它在DateTime之上增加了精度的设置，例如：

```sql
CREATE TABLE Datetime64_TEST (
    c1 Datetime64(2)    
) ENGINE = Memory

--以字符串形式写入
INSERT INTO Datetime64_TEST VALUES('2019-06-22 00:00:00')
 
SELECT c1, toTypeName(c1) FROM Datetime64_TEST

┌─────────────c1─┬─toTypeName(c1)─────────┐
│ 2019-06-22 00:00:00.00 │ DateTime       │
└────────────────────────┴────────────────┘
```

## Date

Date类型不包含具体的时间信息，只精确到天，它同样也支持字符串形式写入：

```sql
CREATE TABLE Date_TEST (
  c1 Date
) ENGINE = Memory

--以字符串形式写入
INSERT INTO Date_TEST VALUES('2019-06-22')

SELECT c1, toTypeName(c1) FROM Date_TEST

┌─────────c1─┬─toTypeName(c1)────────┐
│ 2019-06-22       │ Date            │
└──────────────────┴─────────────────┘

```

## Date32

[v22.1]: https://clickhouse.com/docs/zh/whats-new/changelog/#clickhouse-release-v22-1-2022-01-18

> 注意: [v22.1]版本中新增支持。

一个日期。 支持与 [Datetime64](#datetime64) 相同的日期范围。 存储自 `1925-01-01` 以来的天数，占用4个字节。 允许将日期存储到`2283-11-11`。

```sql
CREATE TABLE Date32_TEST ( c1 Date32 ) ENGINE = Memory

--以字符串形式写入
INSERT INTO Date32_TEST VALUES('2282-06-22')

SELECT c1, toTypeName(c1) FROM Date32_TEST

┌─────────c1─┬─toTypeName(c1)────────┐
│ 2282-06-22       │ Date32          │
└──────────────────┴─────────────────┘
```
