---
title: 前端axios踩坑
date: 2019-03-22 16:05:46
tags: javascript
id: 1553242000
---
axios 库是前端非常流行的网络请求库，下面是我刚使用的时候遇到的坑

本地开发跨域问题（CORS），首先在服务端要接受跨域
```php
header("Access-Control-Allow-Origin: *");
```

这个时候 axios 在 post/put 的时候会先发起 option 请求。如果服务器不支持，则返回失败。为了避免这个，要避免 json 请求。qs 库可以把 json 解析成 www-form 的样子，即 a=1&b=2

axios 有两个数据参数 params 和 data，params 会被放到 url 里，data 会被放到 body 里

```js
const qs = require('qs')

if(param.method){
  let method = param.method.toUpperCase()
  if(method == 'POST' || method == 'PUT'){
    param.headers = {
        'Content-type': 'application/x-www-form-urlencoded;charset=utf-8'
    }
    param.data = qs.stringify(param.data, {arrayFormat: 'brackets'})
  }

  if(method == 'DELETE'){
    param.paramsSerializer = params => {
        return qs.stringify(params, {arrayFormat: 'brackets'})
    }
  }
}else{
  param.paramsSerializer = params => {
    return qs.stringify(params, {arrayFormat: 'brackets'})
  }
}
axios(param)
```

像上面这样修改后，浏览器就不会报跨域错误了，而且服务端还能获得 form 参数，而不是 json。