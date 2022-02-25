# 连接数据库

```shell
> clickhouse-client -h host -p port
```

**常见参数:**

- --host/-h：服务端的地址，默认值为localhost。

    ```shell
    > clickhouse-client -h 10.37.129.10
    ```

- --port：服务端的TCP端口，默认值为9000。
- --user/-u：登录的用户名，默认值为default。
- --password：登录的密码，默认值为空。
- --database/-d：登录的数据库，默认值为default。
- --query/-q：只能在非交互式查询时使用，用于指定SQL语句。
- --multiquery/-n：在非交互式执行时，允许一次运行多条SQL语句，多条语句之间以分号间隔。
- --time/-t：在非交互式执行时，会打印每条SQL的执行时间，例如

    ```shell
    > clickhouse-client -h 10.37.129.10 -n -t --query="SELECT 1;SELECT 2;"
    1
    0.002
    2
    0.001
    ```

- 完整的参数列表，可以通过--help查阅。
