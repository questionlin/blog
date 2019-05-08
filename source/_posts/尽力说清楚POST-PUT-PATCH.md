---
title: 尽力说清楚POST/PUT/PATCH
date: 2019-05-08 11:39:06
tags: http
id: 1557286782
---
# 首先要先区分 form-data 和 x-www-form-urlencoded
在 postman 里面，我们可以看到 form-data 的源代码类似
```
POST /api/order?ab=ab HTTP/1.1
Host: localhost:8081
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
cache-control: no-cache
Postman-Token: 07d87f1e-94fd-4656-881c-898cfc66be36

Content-Disposition: form-data; name="goodsId"

1

Content-Disposition: form-data; name="num"

1

Content-Disposition: form-data; name="skuMapIndex"

黄色>M

Content-Disposition: form-data; name="addressId"

1

------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

x-www-form-urlencoded 的源代码类似
```
POST /api/order?ab=ab HTTP/1.1
Host: localhost:8081
Content-Type: application/x-www-form-urlencoded
cache-control: no-cache
Postman-Token: 92f20568-aad7-445d-aaae-e949db87c53d
%08apiToken=ljjljja=b
```

x-www-form-urlencoded 优点：
1. 使用&连接的k=v字符串，更简洁

缺点：
1. 使用utf-8编码，中文长度变长
2. 不支持文件（2进制内容）

# POST PUT 和 PATCH
标准中 post 用来更新全部内容，put 和 patch 用来更新部分资源。其中 put 是幂等的，patch 不是幂等。

但是实际项目中，即使 post 也不会用来更新全部内容，比如 created_time 就一定是服务器端生成，而不是客户端上传。因此这三者更实用上的区别是：**PUT、PATCH 不支持 form-data**