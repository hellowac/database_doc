# 用户定义

使用users标签可以配置自定义用户。如果打开`user.xml`配置文件，会发现已经默认配置了`default`用户，在此之前的所有示例中，一直使用的正是这个用户。

定义一个新用户，必须包含以下几项属性。

1. **username**

    username用于指定登录用户名，这是全局唯一属性。该属性比较简单，这里就不展开介绍了。

2. **password**

    password用于设置登录密码，支持明文、SHA256加密和double_sha1加密三种形式，可以任选其中一种进行设置。现在分别介绍它们的使用方法。

    （1）明文密码：在使用明文密码的时候，直接通过password标签定义，例如下面的代码。

    ```xml
    <password>123</password>
    ```

    如果password为空，则表示免密码登录：

    ```xml
    <password></password>
    ```

    （2）SHA256加密：在使用SHA256加密算法的时候，需要通过password_sha256_hex标签定义密码，例如下面的代码。

    ```xml
    <password_sha256_hex>a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3</password_sha256_hex>
    ```

    可以执行下面的命令获得密码的加密串，例如对明文密码123进行加密：

    ```shell
    # echo -n 123 | openssl dgst -sha256
    (stdin)= a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3
    ```

    （3）double_sha1加密：在使用double_sha1加密算法的时候，则需要通过password_double_sha1_hex标签定义密码，例如下面的代码。

    ```xml
    <password_double_sha1_hex>23ae809ddacaf96af0fd78ed04b6a265e05aa257</password_double_sha1_hex>
    ```

    可以执行下面的命令获得密码的加密串，例如对明文密码123进行加密：

    ```shell
    # echo -n 123 | openssl dgst -sha1 -binary | openssl dgst -sha1
    (stdin)= 23ae809ddacaf96af0fd78ed04b6a265e05aa257
    ```

3. **networks**

    networks表示被允许登录的网络地址，用于限制用户登录的客户端地址，关于这方面参考[权限管理](../clickhouse/manage_authority.md)

4. **profile**

    用户所使用的profile配置，直接引用相应的名称即可，例如：

    ```xml
    <default>
        <profile>default</profile>
    </default>
    ```

    该配置的语义表示，用户default使用了名为default的profile。

5. **quota**

    `quota`用于设置该用户能够使用的资源限额，可以理解成一种熔断机制。关于这方面参考[熔断机制](../clickhouse/manage_circuit.md)

    现在用一个完整的示例说明用户的定义方法。首先创建一个使用明文密码的用户user_plaintext：

    ```xml
    <yandex>
        <profiles>
            ……
        </profiles>
        <users>
            <default><!—默认用户 -->
            ……
            </default>
            <user_plaintext>
            <password>123</password>
            <networks>
                <ip>::/0</ip>
            </networks>
            <profile>normal_1</profile>
            <quota>default</quota>
            </user_plaintext>
    ```

    由于配置了密码，所以在登录的时候需要附带密码参数：

    ```shell
    # clickhouse-client -h 10.37.129.10 -u user_plaintext --password 123
    Connecting to 10.37.129.10:9000 as user user_plaintext.
    ```

    接下来是两组使用了加密算法的用户，首先是用户user_sha256：

    ```xml
    <user_sha256>
        <!-- echo -n 123 | openssl dgst -sha256 !-->
        <password_sha256_hex>a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3</password_sha256_hex>
        <networks>
            <ip>::/0</ip>
        </networks>
        <profile>default</profile>
        <quota>default</quota>
    </user_sha256>
    ```

    然后是用户user_double_sha1：

    ```xml
    <user_double_sha1>
        <!-- echo -n 123 | openssl dgst -sha1 -binary | openssl dgst -sha1 !-->
        <password_double_sha1_hex>23ae809ddacaf96af0fd78ed04b6a265e05aa257</pass-word_double_sha1_hex>
        <networks>
            <ip>::/0</ip>
        </networks>
        <profile>default</profile>
        <quota>limit_1</quota>
    </user_double_sha1>
    ```

    这些用户在登录时同样需要附带加密前的密码，例如：

    ```shell
    # clickhouse-client -h 10.37.129.10 -u user_sha256 --password 123
    Connecting to 10.37.129.10:9000 as user user_sha256.
    ```
