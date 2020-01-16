---
title: swoole扫雷
date: 2020-01-15 17:45:27
tags: php
id: 1579146320
---
# http/client
```php
new Swoole\Coroutine\Http\Client($url, $port, $ssl);
```
$url 参数不带协议（http/https），如果请求 https，$port=443，$ssl=true

# http/request
通过 $request->server['request_method] 判断请求是 get 还是 post 还是别的

# 跨域
```php
$response->header('Access-Control-Allow-Origin','*');
```

# 协程锁
swoole 每次只有一个协程在运行，所以不用加锁
参考：  
https://wiki.swoole.com/wiki/page/p-differences_with_go.html

# 打印 swoole 配置
php --ri swoole