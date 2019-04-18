---
title: http2与服务端配置
date: 2018-09-29 11:19:57
tags: http
id: 1538191275
---
# http2 与 http1 的主要区别
在于以下几点：
- http2 使用一小块一小块的二进制帧传输
- http2 文件可以设置优先级
- http2 header 部分会被压缩
- http2 服务端可以主动推送文件

# 服务端配置
## nginx 配置
http2 要求服务器支持 https，在之前的文章[《HTTPS服务器配置》](/posts/1528164470)中已经介绍了怎么配置 https，下面给一个比较常用的 nginx http2 配置
```nginx
# http 跳转到 https
server {
    listen         80;
    listen    [::]:80;
    server_name    example.com;
    return         301 https://$server_name$request_uri;
}

server {
    # 在这里加上 http2 就可以开启 http2 支持
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;

    server_name example.com;

    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    ssl_session_timeout  5m;

    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers   on;

    location / {
        try_files $uri $uri/ =404;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        # 这里是服务端推送文件配置，如果没有文件需要推送，可以不加
        http2_push /style.css;
        http2_push /example.png;
    }
}
```

## PHP 服务端推送
推送文件是要经常改动的，但我们又不希望经常改动 nginx 配置文件，这时候可以通过后端程序实现。做法是在 http 头部添加 link 字段：
> Link: </styles.css>; rel=preload; as=style
> Link: </example.png>; rel=preload; as=image

对应的 PHP 代码是：
```php
<?php
header("Link: <{$uri}>; rel=preload; as=image",false);
```

相应 nginx 配置添加
```nginx
server {
    listen 443 ssl http2;
    # ...
    location = / {
        http2_push_preload on;
        # nginx 接 swoole 时需要加这行转发，接 php-fpm 时不需要
        proxy_pass http://upstream;
    }
}
```



各种语言可以参考 [Go](https://ops.tips/blog/nginx-http2-server-push/), [Node](https://blog.risingstack.com/node-js-http-2-push/), [PHP](https://blog.cloudflare.com/using-http-2-server-push-with-php/) 的实现示范。

----------------------
参考资料：  
nginx容器教程：http://www.ruanyifeng.com/blog/2018/02/nginx-docker.html  
How To Set Up Nginx with HTTP/2 Support on Ubuntu 16.04：https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-with-http-2-support-on-ubuntu-16-04  
Using HTTP/2 Server Push with PHP：https://blog.cloudflare.com/using-http-2-server-push-with-php/  
HTTP/2 服务器推送（Server Push）教程：http://www.ruanyifeng.com/blog/2018/03/http2_server_push.html