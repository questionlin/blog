---
title: Elasticsearch基本操作
date: 2018-07-13 15:29:30
tags:
- elasticsearch
- 运维
id: 1531467381
---
这篇文章的意义一来是给写 API 一个参考，二来是紧急情况控制台排查。因为浏览器和 postman 都不支持 GET 后面跟数据，所以所有命令都以 curl 形式给出。这篇文章里 url 里的**loc{}内的内容是可以替换的**

## 集群里文档数量
```sh
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

## 集群健康
```sh
curl -XGET 'http://localhost:9200/_cluster/health'
```
返回以下数据
```json
{
    "cluster_name": "elasticsearch",
    "status": "green",
    "timed_out": false,
    "number_of_nodes": 1,
    ...
}
```
其中 status 是我们最感兴趣的，它有一下含义
颜色 | 意义
-|-
green | 所有主要分片和复制分片都可用
yallow | 所有主要分片可用，但不是所有复制分片都可用
red | 不是所有主要分片都可用

## 添加索引
```sh
curl -XPUT 'http://localhost:9200/{index}' -d '
{
    "key1": "value1",
    "key2": "value2"
}
'
```

-------------------------------
参考资料：《Elasticsearch 权威指南》