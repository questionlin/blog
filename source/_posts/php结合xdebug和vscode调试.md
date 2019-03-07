---
title: php结合xdebug和vscode调试
date: 2019-03-07 11:46:36
tags: php
id: 1551930448
---
# 准备工作
1. php 安装 xdebug 扩展
```sh
pecl install xdebug
```
添加配置
```
[xdebug]
zend_extension=xdebug.so
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.remote_port = 9001
; 默认端口号是9000，跟php-fpm冲突
```

2. vscode 安装 PHP Debug 扩展
点菜单的调试-打开配置-PHP，在配置里添加
```
"configurations": [
    {
        "name": "Listen for XDebug",
        "type": "php",
        "request": "launch",
        "port": 9001
    },
```

# 使用
点左边菜单的小虫子，然后就可以正常的调试了
