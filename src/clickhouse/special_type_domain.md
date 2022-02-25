# Domain

[IPv4NumToString]: https://clickhouse.com/docs/en/sql-reference/functions/ip-address-functions/#ipv4numtostringnum
[IPv6NumToString]: https://clickhouse.com/docs/en/sql-reference/functions/ip-address-functions/#ipv6numtostringx

域名类型分为IPv4和IPv6两类，本质上它们是对整型和字符串的进一步封装。IPv4类型是基于UInt32封装的。

```sql
CREATE TABLE IP4_TEST (
  url String,
  ip IPv4
) ENGINE = Memory;

INSERT INTO IP4_TEST VALUES ('www.nauu.com','192.0.0.0')

SELECT url , ip ,toTypeName(ip) FROM IP4_TEST

┌─url──────────┬─────ip────┬─toTypeName(ip)───────┐
│ www.nauu.com │ 192.0.0.0 │     IPv4             │
└──────────────┴───────────┴──────────────────────┘
```

直接使用字符串不就行了吗？

- （1）出于便捷性的考量，例如IPv4类型支持格式检查，格式错误的IP数据是无法被写入的，例如：

    ```sql
    INSERT INTO IP4_TEST VALUES ('www.nauu.com','192.0.0')
    Code: 441. DB::Exception: Invalid IPv4 value.
    ```

- （2）出于性能的考量，同样以IPv4为例，IPv4使用UInt32存储，相比String更加紧凑，占用的空间更小，查询性能更快。IPv6类型是基于FixedString(16)封装的，它的使用方法与IPv4别无二致.

在使用Domain类型的时候还有一点需要注意，虽然它从表象上看起来与String一样，但Domain类型并不是字符串，所以它不支持隐式的自动类型转换。

如果需要返回IP的字符串形式，则需要显式调用[IPv4NumToString]或[IPv6NumToString]函数进行转换。
