# Runtime

在`4.1.0`版本中，底层增加一个新的特性，可以在运行时动态将基于`php_stream`实现的扩展、`PHP`网络客户端代码一键协程化。底层替换了`ZendVM` `Stream`的函数指针，所有使用`php_stream`进行`socket`操作均变成协程调度的异步`IO`。

目前有`PHP`原生`Redis`、`PDO`、`MySQLi`协程化的支持。

> `4.1`版本仅支持`tcp`和`unix`两种`stream`类型  
> `4.2`版本增加了对`udp`、`udg`、`unix`、`ssl`、`tls`类型的支持  
> `4.2.3`版本以前存在FILE_HOOK覆盖include/require的BUG, 请通过`Swoole\Runtime::enableCoroutine(true, SWOOLE_HOOK_ALL ^ SWOOLE_HOOK_FILE);`的方式屏蔽file hook


函数原型
----
```php
function Runtime::enableCoroutine(bool $enable = true, int $flags = SWOOLE_HOOK_ALL);
```

* `$enable`：打开或关闭协程
* `$flags`：选择要`Hook`的类型，可以多选，默认为全选。仅在`$enable = true`时有效

> `$flags`参数在`4.2`或更高版本可用，请参考：[开关选项](https://wiki.swoole.com/wiki/page/993.html)

可用列表
----
* `redis`扩展
* 使用`mysqlnd`模式的`pdo`、`mysqli`扩展，如果未启用`mysqlnd`将不支持协程化
* `soap`扩展
* `file_get_contents`、`fopen`
* `stream_socket_client` (predis)
* `stream_socket_server`
* `fsockopen`

不可用列表
----
* `mysql`：底层使用`libmysqlclient`
* `curl`：底层使用`libcurl` （即不能使用`CURL`驱动的`Guzzle`）
* `mongo`：底层使用`mongo-c-client`
* `pdo_pgsql`
* `pdo_ori`
* `pdo_odbc`
* `pdo_firebird`

使用实例
----

```php
Swoole\Runtime::enableCoroutine();

go(function () {
    $redis = new redis;
    $retval = $redis->connect("127.0.0.1", 6379);
    var_dump($retval, $redis->getLastError());
    var_dump($redis->get("key"));
    var_dump($redis->set("key", "value2"));
    var_dump($redis->get("key"));
    $redis->close();
});
```

方法摆放位置
---
调用方法后当前进程内全局生效, 一般放在整个项目最开头以获得100%覆盖的效果, 协程内外会自动切换模式, 不影响PHP原生环境使用.

注意: 不建议放在onRequest等回调中开启, 会多次调用造成不必要的调用开销.

