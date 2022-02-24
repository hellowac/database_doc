- [数值类型](#数值类型)
  - [Int](#int)
  - [Float](#float)
  - [Decimal](#decimal)

# 数值类型

数值类型分为`整数`、`浮点数`和`定点数`三类.

## Int

参考官网: <https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint/>

创建表时，可以为整数设置类型参数 (例如. TINYINT(8), SMALLINT(16), INT(32), BIGINT(64)), 但 ClickHouse 会忽略它们.

有符号

| 名称   | 大小(字节) | 范围                                             | 常用     |
| ------ | ---------- | ------------------------------------------------ | -------- |
| Int8   | 1          | -128 - 127                                       | Tinyint  |
| Int16  | 2          | -32768 - 32767                                   | Smallint |
| Int32  | 4          | -2147483648 - 2147483647                         | Int      |
| Int64  | 8          | -9223372036854775808 - 9223372036854775807       | Bigint   |
| Int128 | 16         | -1701411834604692317... - 1701411834604692317... | -        |
| Int256 | 32         | -5789604461865809771... - 5789604461865809771... | -        |

无符号

| 名称    | 大小(字节) | 范围                                           | 常用              |
| ------- | ---------- | ---------------------------------------------- | ----------------- |
| UInt8   | 1          | 0 - 255                                        | Tinyint Unsigned  |
| UInt16  | 2          | 0 - 65535                                      | Smallint Unsigned |
| UInt32  | 4          | 0 - 4294967295                                 | Int Unsigned      |
| UInt64  | 8          | 0 - 18446744073709551615                       | Bigint Unsigned   |
| UInt128 | 16         | 0 - 340282366920938463463374607431768211455    | -                 |
| UInt256 | 32         | 0 - 115792089237316195423570985008687907853... | -                 |

## Float

参考官网: <https://clickhouse.com/docs/zh/sql-reference/data-types/float/>

与整数类似，ClickHouse直接使用`Float32`和`Float64`代表单精度浮点数以及双精度浮点数

| 名称    | 大小(字节) | 有效经度(位数) | 常用   |
| ------- | ---------- | -------------- | ------ |
| Float32 | 4          | 7              | Float  |
| Float64 | 8          | 16             | Double |

ClickHouse的浮点数支持正无穷、负无穷以及非数字的表达方式。

- Inf – 正无穷

    ```sql
    >) SELECT 0.5 / 0
    ┌─divide(0.5, 0)─┐
    │            inf │
    └────────────────┘
    ```

- -Inf – 负无穷

    ```sql
    >) SELECT -0.5 / 0
    ┌─divide(-0.5, 0)─┐
    │            -inf │
    └─────────────────┘
    ```

- NaN – 非数字

    ```sql
    >) SELECT 0 / 0
    ┌─divide(0, 0)──┐
    │            nan│
    └───────────────┘
    ```

## Decimal

参考官网: <https://clickhouse.com/docs/zh/sql-reference/data-types/decimal/>

更高精度的数值运算，则需要使用定点数。ClickHouse提供了Decimal32、Decimal64和Decimal128三种精度的定点数。可以通过两种形式声明定点：简写方式有Decimal32(S)、Decimal64(S)、Decimal128(S)三种，原生方式为Decimal(P,S)，其中：

- P代表精度，决定总位数（整数部分+小数部分），取值范围是1～38；
- S代表规模，决定小数位数，取值范围是0～P。

对应关系：

| 名称          | 有效声明             | 范围                                               |
| ------------- | -------------------- | -------------------------------------------------- |
| Decimal32(S)  | Decimal(1 ~ 9 , S)   | -1 * 10<sup>^</sup>(9-S) 至  10<sup>^</sup>(9-S)   |
| Decimal64(S)  | Decimal(10 ~ 18 , S) | -1 * 10<sup>^</sup>(18-S) 至  10<sup>^</sup>(18-S) |
| Decimal128(S) | Decimal(19 ~ 38 , S) | -1 * 10<sup>^</sup>(38-S) 至  10<sup>^</sup>(38-S) |

由于现代CPU不支持128位数字，因此 `Decimal128` 上的操作由软件模拟。所以 `Decimal128` 的运算速度明显慢于 `Decimal32`/`Decimal64`。

**对Decimal的二进制运算导致更宽的结果类型（无论参数的顺序如何）。**

- Decimal64(S1) \<op\> Decimal32(S2) -> Decimal64(S)
- Decimal128(S1) \<op\> Decimal32(S2) -> Decimal128(S)
- Decimal128(S1) \<op\> Decimal64(S2) -> Decimal128(S)

**精度变化的规则：**

- 加法，减法：S = max(S1, S2)。
- 乘法：S = S1 + S2。
- 除法：S = S1。

例如`toDecimal64(2,4)`与`toDecimal32(2,2)`相加或相减后`S=4`：

```shell
# 相加
:) SELECT toDecimal64(2,4) + toDecimal32(2,2)
 
┌─plus(toDecimal64(2, 4), toDecimal32(2, 2))─┐
│ 4.0000                                     │ # 保留四位
└────────────────────────────────────────────┘

# 相减
:) SELECT toDecimal32(4,4) - toDecimal64(2,2)

┌─minus(toDecimal32(4, 4), toDecimal64(2, 2))┐
│ 2.0000                                     │ # 保留四位
└────────────────────────────────────────────┘

# toDecimal64(2,4)与toDecimal32(2,2)相乘后S=4+2=6
:) SELECT toDecimal64(2,4) * toDecimal32(2,2)

┌─multiply(toDecimal64(2, 4), toDecimal32(2, 2))┐
│ 4.000000                                      │
└───────────────────────────────────────────────┘

# toDecimal64(2,4)与toDecimal32(2,2)相除后S=4
:) SELECT toDecimal64(2,4) / toDecimal32(2,2)

┌─divide(toDecimal64(2, 4), toDecimal32(2, 2))─┐
│  1.0000                                      │
└──────────────────────────────────────────────┘

```
