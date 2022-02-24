- [字符串类型](#字符串类型)
  - [String](#string)
  - [FixedString](#fixedstring)
  - [UUID](#uuid)

# 字符串类型

字符串类型可以细分为`String`、`FixedString`和`UUID`三类

## String

字符串由String定义，长度不限。因此在使用String的时候无须声明大小。它完全代替了传统意义上数据库的Varchar、Text、Clob和Blob等字符类型。String类型不限定字符集，因为它根本就没有这个概念，所以可以将任意编码的字符串存入其中。但是为了程序的规范性和可维护性，在同一套程序中应该遵循使用统一的编码，例如“统一保持UTF-8编码”就是一种很好的约定。

## FixedString

FixedString类型和传统意义上的Char类型有些类似，对于一些字符有明确长度的场合，可以使用固定长度的字符串。定长字符串通过FixedString(N)声明，其中N表示字符串长度。但与Char不同的是，FixedString使用null字节填充末尾字符，而Char通常使用空格填充。比如在下面的例子中，字符串‘abc’虽然只有3位，但长度却是5，因为末尾有2位空字符填充：

```sql
:) SELECT toFixedString('abc',5) , LENGTH(toFixedString('abc',5)) AS LENGTH

┌─toFixedString('abc', 5)─┬─LENGTH──────┐
│ abc                     │ 5           │
└─────────────────────────┴─────────────┘
```

## UUID

UUID是一种数据库常见的主键类型，在ClickHouse中直接把它作为一种数据类型。UUID共有32位，它的格式为8-4-4-4-12。如果一个UUID类型的字段在写入数据时没有被赋值，则会依照格式使用0填充，例如：

```sql

CREATE TABLE UUID_TEST (
  c1 UUID,
  c2 String
) ENGINE = Memory;

--第一行UUID有值
INSERT INTO UUID_TEST SELECT generateUUIDv4(),'t1'
--第二行UUID没有值
INSERT INTO UUID_TEST(c2) VALUES ('t2')

:) SELECT * FROM UUID_TEST
┌─────────────────────c1─┬─c2─┐
│ f36c709e-1b73-4370-a703-f486bdd22749 │ t1 │
└───────────────────────┴────┘
┌─────────────────────c1─┬─c2─┐
│ 00000000-0000-0000-0000-000000000000 │ t2 │
└───────────────────────┴────┘
```
