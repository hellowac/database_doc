# 查询权限

查询权限是整个权限体系的第二层防护，它决定了一个用户能够执行的查询语句。查询权限可以分成以下五类：

- **读权限**：包括SELECT、EXISTS、SHOW和DESCRIBE查询。

- **写权限**：包括INSERT和OPTIMIZE查询。

- **设置权限**：包括SET查询。

- **DDL权限**：包括CREATE、DROP、ALTER、RENAME、ATTACH、DETACH和TRUNCATE查询。

- **其他权限**：包括KILL和USE查询，任何用户都可以执行这些查询。

上述这五类权限，通过以下两项配置标签控制：

- （1）**readonly**：读权限、写权限和设置权限均由此标签控制，它有三种取值。

  - **当取值为0时**，不进行任何限制（默认值）。
  - **当取值为1时**，只拥有读权限（只能执行SELECT、EXISTS、SHOW和DESCRIBE）。
  - **当取值为2时**，拥有读权限和设置权限（在读权限基础上，增加了SET查询）。

- （2）**allow_ddl**：DDL权限由此标签控制，它有两种取值。

  - **当取值为0时**，不允许DDL查询。

  - **当取值为1时**，允许DDL查询（默认值）。

现在继续用一个示例说明。与刚才的配置项不同，`readonly`和`allow_ddl`需要定义在用户profiles中，例如：

```xml
<profiles>        
  <normal> <!-- 只有read读权限-->
    <readonly>1</readonly>
    <allow_ddl>0</allow_ddl>
  </normal>
  <normal_1> <!-- 有读和设置参数权限-->
    <readonly>2</readonly>
    <allow_ddl>0</allow_ddl>
  </normal_1>
```

继续在先前的profiles配置中追加了两个新角色。其中，normal只有读权限，而normal_1则有读和设置参数的权限，它们都没有DDL查询的权限。

再次修改用户的user_normal的定义，将它的profile设置为刚追加的normal，这意味着该用户只有读权限。现在开始验证权限的设置是否生效。使用user_normal登录后尝试写操作：

```sql
--登录
# clickhouse-client -h 10.37.129.10 -u user_normal

--写操作
:) INSERT INTO TABLE test_ddl VALUES (1)

DB::Exception: Cannot insert into table in readonly mode.
```

可以看到，写操作如期返回了异常。接着执行DDL查询，会发现该操作同样会被限制执行：

```sql
:) CREATE DATABASE test_3

DB::Exception: Cannot create database in readonly mode.
```

至此，权限设置已然生效。
