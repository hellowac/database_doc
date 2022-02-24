# 内置工具

## clickhouse-local

clickhouse-local可以独立运行大部分SQL查询，不需要依赖任何ClickHouse的服务端程序，它可以理解成是ClickHouse服务的单机版微内核，是一个轻量级的应用程序。

clickhouse-local只能够使用File表引擎（关于表引擎的更多介绍在后续章节展开），它的数据与同机运行的ClickHouse服务也是完全隔离的，相互之间并不能访问。

clickhouse-local是非交互式运行的，每次执行都需要指定数据来源，例如通过stdin标准输入，以echo打印作为数据来源：

```shell
> echo -e "1\n2\n3" | clickhouse-local -q "CREATE TABLE test_table (id Int64)  ENGINE = File(CSV, stdin); SELECT id FROM test_table;"
1
2
3
```

也可以借助操作系统的命令，实现对系统用户内存用量的查询：

```shell
> ps aux | tail -n +2 | awk '{ printf("%s\t%s\n", $1, $4) }' | clickhouse-local -S "user String, memory Float64" -q "SELECT user, round(sum(memory), 2) as memoryTotal FROM table GROUP BY user ORDER BY memoryTotal DESC FORMAT Pretty"
┏----------┳-----------------------------┓
┃ user     ┃        memoryTotal          ┃
┡----------╇-----------------------------┩
│ nauu     │        42.7                 │
├──────────┼──────────----------------───┤
│ root     │        20.4                 │
├──────────┼─────────----------------────┤
│ clickho+ │         1.8                 │
└──────────┴────────----------------─────┘
```

**核心参数**：

- （1）-S/--structure：表结构的简写方式，例如以下两种声明的效果是相同的。

    ```shell
    --使用-S简写
    clickhouse-local -S "id Int64"
    --使用DDL
    clickhouse-local -q "CREATE TABLE test_table (id Int64) ENGINE = File(CSV, stdin)"
    ```

- （2）-N/--table：表名称，默认值是table，例如下面的代码。

    ```shell
    clickhouse-local -S "id Int64" -N "test_table" -q "SELECt id FROM test_table"
    ```

- （3）-if/--input-format：输入数据的格式，默认值是TSV，例如下面的代码。

    ```shell
    echo -e "1\n2\n3" | clickhouse-local -S "id Int64" -if "CSV" -N "test_table"
    ```

- （4）-f/--file：输入数据的地址，默认值是stdin标准输入。

- （5）-q/--query：待执行的SQL语句，多条语句之间以分号间隔。

完整的参数列表可以通过--help查阅。

## clickhouse-benchmark

clickhouse-benchmark是基准测试的小工具，它可以自动运行SQL查询，并生成相应的运行指标报告，例如执行下面的语句启动测试：

```shell
> echo "SELECT * FROM system.numbers LIMIT 100" | clickhouse-benchmark -i 5
Loaded 1 queries.

# 执行之后，按照指定的参数该查询会执行5次：
Queries executed: 5.  

# 执行完毕后，会出具包含QPS、RPS等指标信息的报告：
127.0.0.1:9000, queries 5, QPS: 812.189, RPS: 81218.868, MiB/s: 0.620, result RPS: 81218.868, result MiB/s: 0.620.

# 还会出具各百分位的查询执行时间：
0.000%          0.001 sec.
10.000%         0.001 sec.
20.000%         0.001 sec.
30.000%         0.001 sec.
40.000%         0.001 sec.
50.000%         0.001 sec.
60.000%         0.001 sec.
70.000%         0.001 sec.
80.000%         0.001 sec.
90.000%         0.002 sec.
95.000%         0.002 sec.
99.000%         0.002 sec.
99.900%         0.002 sec.
99.990%         0.002 sec.

# 可以指定多条SQL进行测试，此时需要将SQL语句定义在文件中：
> cat ./multi-sqls 
SELECT * FROM system.numbers LIMIT 100
SELECT * FROM system.numbers LIMIT 200

# 在multi-sqls文件内定义了两条SQL，按照定义的顺序它们会依次执行：
> clickhouse-benchmark -i 5 < ./multi-sqls
Loaded 2 queries.
……
```

**核心参数**:

- （1）-i/--iterations：SQL查询执行的次数，默认值是0。

- （2）-c/--concurrency：同时执行查询的并发数，默认值是1。

- （3）-r/--randomize：在执行多条SQL语句的时候，按照随机顺序执行，例如:

    ```shell
    > clickhouse-benchmark -r 1 -i 5  < ./multi-sqls
    ```

- （4）-h/--host：服务端地址，默认值是localhost。clickhouse-benchmark支持对比测试，此时需要通过此参数声明两个服务端的地址，例如:

    ```shell
    > echo "SELECT * FROM system.numbers LIMIT 100" | clickhouse-benchmark -i 5 -h localhost -h localhost
    Loaded 1 queries.

    Queries executed: 5.

    # 第一个服务
    127.0.0.1:9000, queries 2, QPS: 878.703, RPS: 87870.258, MiB/s: 0.670, result RPS: 87870.258, result MiB/s: 0.670.
    # 第二个服务
    127.0.0.1:9000, queries 3, QPS: 748.210, RPS: 74820.972, MiB/s: 0.571, result RPS: 74820.972, result MiB/s: 0.571.

    # 在对比测试中，clickhouse-benchmark会通过抽样的方式比较两组查询指标的差距，在默认的情况下，置信区间为99.5%：
    0.000%          0.001 sec.      0.001 sec.
    10.000%         0.001 sec.      0.001 sec.
    20.000%         0.001 sec.      0.001 sec.
    30.000%         0.001 sec.      0.001 sec.
    40.000%         0.001 sec.      0.001 sec.
    50.000%         0.001 sec.      0.001 sec.
    60.000%         0.001 sec.      0.001 sec.
    70.000%         0.001 sec.      0.001 sec.
    80.000%         0.001 sec.      0.002 sec.
    90.000%         0.001 sec.      0.002 sec.
    95.000%         0.001 sec.      0.002 sec.
    99.000%         0.001 sec.      0.002 sec.
    99.900%         0.001 sec.      0.002 sec.
    99.990%         0.001 sec.      0.002 sec.

    No difference proven at 99.5% confidence
    ```

- （5）--confidence：设置对比测试中置信区间的范围，默认值是5(99.5%)，它的取值范围有0(80%)、1(90%)、2(95%)、3(98%)、4(99%)和5(99.5%)。
