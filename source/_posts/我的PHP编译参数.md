---
title: 我的PHP编译参数
date: 2018-12-12 11:10:08
tags: php
id: 1544584259
---
由于 PHP 高度依赖第三方插件，导致编译 PHP 变成一个难题。线上环境要求一致，不允许使用 yum 或 apt 来安装，只能在编译环境编译好后复制到线上，因此每一个 PHP 程序员都应该有一份自己的编译参数。这篇文章介绍我的参数。

```sh
# 首先编译环境要先安装好
$ yum install libcurl-devel libjpeg-turbo-devel libpng-devel libwebp-devel libxml2-devel libXpm-devel openssl-devel zlib-devel

# 然后开始编译安装
# --prefix 是安装的目录
$ ./configure --prefix=/data/server/php \
--enable-fpm \
--enable-ftp \
--enable-pcntl \
--enable-mbstring \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-shmop \
--enable-pdo \
--with-curl \
--with-openssl \
--with-jpeg-dir \
--with-webp-dir \
--with-xpm-dir \
--with-gd \
--with-pdo-mysql \
--with-zlib \

$ make && make install
$ pecl install event
$ pecl install redis
```