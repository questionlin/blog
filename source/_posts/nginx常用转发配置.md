---
title: nginx常用转发配置
date: 2019-12-21 10:41:21
tags: 最佳实践
id: 1576906217
---
# http 跳转 https
```conf
server {
    if ($host = www.ljj.pub) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
}
server {
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    server_name www.ljj.pub;
}
```
第一个 server 配置是 certbot 自动生成的，目的是将所有 http://www.ljj.pub 的请求转发到 https://www.ljj.pub。注意第二个 server 配置里面不可以有 listen 80; 否则会覆盖上面那个，导致转发不执行。

# 移动和PC端跳转不同页面
```conf
server {
    if ($http_user_agent ~* 'ipad|iphone|android') {
        return 301 https://m.ljj.pub;
    }
}
```
这里的匹配符号有
- = 严格匹配
- ~ 区分大小写匹配（可用正则）
- ~* 不区分大小写匹配（可用正则）
- !~ 区分大小写不匹配
- !~* 不区分大小写不匹配
- ^~ 如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式
-----------------------------
参考：
[nginx匹配](https://www.cnblogs.com/duhuo/p/8323812.html)    
[在线生成配置的工具](https://www.digitalocean.com/community/tools/nginx)