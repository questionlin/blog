---
title: 手把手配置cdn
date: 2019-10-31 09:47:51
tags: 最佳实践
id: 1572486507
---
# 初始配置
通常 cdn 服务商会让我们配置三个内容
1. 域名
2. 源站
3. 缓存内容规则

比如我的网站已经有了 www.ljj.pub，需要 img.ljj.pub 作为缓存域名。那么①里的域名应该填入 img.ljj.pub。②里的源站填入 www.ljj.pub。这时 nginx 不需要配置 img 域名。

cdn 默认的缓存规则会缓存所有内容，包括动态页面的内容。我需要规则更清楚，因此我设置了按文件类型，.jpg;.jpeg;.png;.svg;.js;.css。过期时间可以自己定。

配置好之后服务商会给一个他们自己的域名，例如 img.ljj.pub.xxx.cloud.cn。**注意这个域名是不能直接访问的**，应该去自己域名的配置页，设置 img.ljj.pub CNAME 指向 img.ljj.pub.xxx.cloud.cn。几分钟后就可以用 img.ljj.pub 访问了。

# https
新网站 https 是标配，我们自己需要我们自己给出证书。不管是从云服务商申请的，还是自己从证书服务商申请的。快过期的时候都要手动更新。我选择使用 certbot 申请，因为每年自动更新本地证书。

首先因为我的 www 和 img 同源，所以我的 nginx 已经配置了
> server_name www.ljj.pub img.ljj.pub;

然后执行

```sh
$ certbot-auto certonly --nginx
```
这个命令只申请证书，不配置 nginx。

然后把得到的 fullchain.pem 和 private.pem 里的内容复制到 cdn 配置里面，就可以了。

这样过一会儿，就能访问 https://img.ljj.pub 了。