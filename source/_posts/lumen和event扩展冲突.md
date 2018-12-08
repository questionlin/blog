---
title: lumen和event扩展冲突
date: 2018-12-08 15:44:55
tags: php
id: 1544255132
---
如果系统里安装了 event 扩展，Lumen 在使用 artisan 或者 bootstrap/app.php 里打开了 $app->withFacades()，那么系统会报
> Cannot declare class Event, because the name is already in use
Laravel 不知道会不会报错，估计也会。

这时候在 Console/Kernel.php 里加入
```php
protected $aliases = false;
```
就可以了。代价是不能使用 Facades 了。