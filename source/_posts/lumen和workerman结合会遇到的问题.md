---
title: lumen和workerman结合会遇到的问题
date: 2018-12-08 15:44:55
tags: php
id: 1544255132
---
# $_FILE
CLI 模式不支持 move_uploaded_file() 函数，因此 $_FILE 内容会和默认的不一样，需要手动处理。参考 http://doc.workerman.net/web-server.html

# 报错
由于 CLI 模式不支持 header() 函数，需要使用 WorkerMan 自带的 Http::header() 代替。我的做法是在
> app/Exceptions/Handler.php

render() 函数加上 Http::header()

# Auth Guard 保留登录信息
如果使用了 Auth 中间件，程序会一直保留登录信息，不管谁登录都返回第一个用户。我的做法是复制默认的 RequestGuard.php 到
> app/Services/Auth/RequestGuard.php

并增加函数
```php
public function logout()
{
    $this->user = null;
}
```

然后在
> app/Providers/AuthServiceProvider.php

里增加这个 guard
```php
public function boot()
{
    $this->app['auth']->extend('api', function($app, $name, array $config) {
        return new RequestGuard(function ($request) {
            if ($request->input('apiToken')) {
                $user = new User;
                return $user->getByApiToken($request->input('apiToken'));
            }
        },
        $app['request'],
        $app['auth']->createUserProvider());
    });
}
```

然后修改
> app/Http/Middleware/Authenticate.php

```php
public function handle($request, Closure $next, $guard = null)
{
    $guard = $this->auth->guard($guard);
    $guard->setRequest($request); // 这里是为了每次请求都重新设置一遍参数
    if ($guard->guest()) {
        return response('Unauthorized.', 401);
    }

    return $next($request);
}
```

最后在每次请求完后执行一次
```php
app('auth')->logout();
```
清理登录信息

# lumen和event扩展冲突
如果系统里安装了 event 扩展，Lumen 在使用 artisan 或者 bootstrap/app.php 里打开了 $app->withFacades()，那么系统会报

> Cannot declare class Event, because the name is already in use

Laravel 不知道会不会报错，估计也会。

这时候在 Console/Kernel.php 里加入
```php
protected $aliases = false;
```
就可以了。代价是不能使用 Facades 了。