# 用户约束

`constraints`标签可以设置一组约束条件，以保障`profile`内的参数值不会被随意修改。约束条件有如下三种规则：

- **Min**：最小值约束，在设置相应参数的时候，取值不能小于该阈值。

- **Max**：最大值约束，在设置相应参数的时候，取值不能大于该阈值。

- **Readonly**：只读约束，该参数值不允许被修改。

现在举例说明：

```xml
<profiles><!-- 配置profiles -->

  <default> <!-- 自定义名称, 默认角色-->
    <max_memory_usage>10000000000</max_memory_usage>
    <distributed_product_mode>allow</distributed_product_mode>

    <constraints><!-- 配置约束-->
      <max_memory_usage>
        <min>5000000000</min>
        <max>20000000000</max>
      </max_memory_usage>
      <distributed_product_mode>
        <readonly/>
      </distributed_product_mode>
    </constraints>

  </default>
```

从上面的配置定义中可以看出，在`default`默认的`profile`内，给两组参数设置了约束。其中，为`max_memory_usage`设置了`min`和`max`阈值；而为`distributed_product_mode`设置了`只读约束`。现在尝试修改`max_memory_usage`参数，将它改为`50`：

```sql
SET max_memory_usage = 50
DB::Exception: Setting max_memory_usage shouldn't be less than 5000000000.
```

可以看到，最小值约束阻止了这次修改。

接着继续修改`distributed_product_mode`参数的取值：

```sql
SET distributed_product_mode = 'deny'
DB::Exception: Setting distributed_product_mode should not be changed.
```

同样，配置约束成功阻止了预期外的修改。

还有一点需要特别明确，在`default`中默认定义的`constraints`约束，将作为默认的全局约束，自动被其他`profile`继承。
