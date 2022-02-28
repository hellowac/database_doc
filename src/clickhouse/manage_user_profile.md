# 用户profile

用户`profile`的作用类似于用户角色。可以预先在`user.xml`中为ClickHouse定义多组`profile`，并为每组`profile`定义不同的配置项，以实现配置的复用。以下面的配置为例：

```xml
<yandex>
  <profiles><!-- 配置profile -->

    <default> <!-- 自定义名称, 默认角色-->
      <max_memory_usage>10000000000</max_memory_usage>
      <use_uncompressed_cache>0</use_uncompressed_cache>
    </default>

    <test1> <!-- 自定义名称, 默认角色-->
      <allow_experimental_live_view>1</allow_experimental_live_view>
      <distributed_product_mode>allow</distributed_product_mode>
    </test1>
  </profiles>
  ……
```

在这组配置中，预先定义了`default`和`test1`两组`profile`。引用相应的`profile`名称，便会获得相应的配置。我们可以在CLI中直接切换到想要的`profile`：

```shell
SET profile = test1
```

或是在定义用户的时候直接引用: 参考[用户定义](../clickhouse/manage_user_define.md)

在所有的`profile`配置中，名称为`default`的`profile`将作为默认的配置被加载，所以它必须存在。如果缺失了名为`default`的`profile`，在登录时将会出现如下错误提示：

```shell
DB::Exception: There is no profile 'default' in configuration file..
```

profile配置支持继承，实现继承的方式是在定义中引用其他的profile名称，例如下面的例子所示：

```xml
<normal_inherit> <!-- 只有read查询权限-->
  <profile>test1</profile>
  <profile>test2</profile>
  <distributed_product_mode>deny</distributed_product_mode>
</normal_inherit>
```

这个名为`normal_inherit`的`profile`继承了`test1`和`test2`的所有配置项，并且使用新的参数值覆盖了`test1`中原有的`distributed_product_mode`配置项。
