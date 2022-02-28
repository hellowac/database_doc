- [访问权限](#访问权限)
  - [1.网络访问权限](#1网络访问权限)
  - [2.数据库与字典访问权限](#2数据库与字典访问权限)

# 访问权限

访问层控制是整个权限体系的第一层防护，它又可进一步细分成两类权限。

## 1.网络访问权限

网络访问权限使用networks标签设置，用于限制某个用户登录的客户端地址，有IP地址、host主机名称以及正则匹配三种形式，可以任选其中一种进行设置。

（1）IP地址：直接使用IP地址进行设置。

```xml
<ip>127.0.0.1</ip>
```

（2）host主机名称：通过host主机名称设置。

```xml
<host>ch5.nauu.com</host>
```

（3）正则匹配：通过表达式来匹配host名称。

```xml
<host>^ch\d.nauu.com$</host>
```

现在用一个示例说明：

```xml
<user_normal>
  <password></password>
  <networks>
    <ip>10.37.129.13</ip>
  </networks>
  <profile>default</profile>
  <quota>default</quota>
</user_normal>
```

用户user_normal限制了客户端IP，在设置之后，该用户将只能从指定的地址登录。此时如果从非指定IP的地址进行登录，例如：

```shell
# 从10.37.129.10登录
> clickhouse-client -u user_normal
```

则将会得到如下错误：

```shell
DB::Exception: User user_normal is not allowed to connect from address 10.37.129.10.
```

## 2.数据库与字典访问权限

在客户端连入服务之后，可以进一步限制某个用户数据库和字典的访问权限，它们分别通过allow_databases和allow_dictionaries标签进行设置。如果不进行任何定义，则表示不进行限制。现在继续在用户user_normal的定义中增加权限配置：

```xml
<user_normal>
  ……
  <allow_databases>
    <database>default</database>
    <database>test_dictionaries</database>
  </allow_databases>
  <allow_dictionaries>
    <dictionary>test_flat_dict</dictionary>
  </allow_dictionaries>
</user_normal>
```

通过上述操作，该用户在登录之后，将只能看到为其开放了访问权限的数据库和字典。
