---
title: mac安装swoole
date: 2019-03-12 10:36:59
tags: php
id: 1552358380
---
在 mac 下 pecl 识别不了 openssl 依赖路径，只能手动编译安装。具体的方法是先下载 swoole 安装包，如果是用 pecl 下载，那么安装包应该在 /tmp/pear/download 里面。

然后执行

```sh
$ brew install openssl #如果需要支持 openssl
$ brew install nghttp2 #如果需要支持 http2
$ cd swoole
$ phpize
$ ./configure --enable-openssl --enable-http2 --enable-sockets --enable-mysqlnd -with-openssl-dir=/usr/local/Cellar/openssl/1.0.2q/
$ make && make install
```

然后把 extension=swoole 写入 php.ini