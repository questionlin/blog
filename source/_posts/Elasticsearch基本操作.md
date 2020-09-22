---
title: Elasticsearch基本操作
date: 2018-07-13 15:29:30
tags:
- elasticsearch
- 运维
id: 1531467381
---
这篇文章的意义一来是给写 API 一个参考，二来是紧急情况控制台排查。因为浏览器和 postman 都不支持 GET 后面跟数据，所以所有命令都以 curl 形式给出。

# 安装
详细的安装过程就不写了，无非安装 java 后安装 elasticsearch。这里提一下，可以使用清华的镜像（https://mirrors.tuna.tsinghua.edu.cn/elasticstack/6.x/yum/6.3.1/elasticsearch-6.3.1.rpm） 。如果你的开发机内存小，修改 /etc/elasticsearch/jvm.option 以下两项：
```
-Xms128m
-Xmx128m
```
128m 是你要分配的内存。

# Index
在 Elasticsearch 里 Index 代表代表一个数据库。

## 查看所有 Index：
```sh
$curl -X GET '127.0.0.1:9200/_cat/indices?v'
```

## 新建 Index
```sh
$curl -X PUT '127.0.0.1:9200/users'
```
返回
```json
{
    "acknowledged": true,
    "shards_acknowledged":true,
    "index":"users"
}
```
返回的数据里有 acknowledged: true 即表示操作成功。这时候查看所有 Index 可以看到 users 库。

## 删除 Index
```sh
$curl -X DELETE '127.0.0.1:9200/users'
```

# Type
Type 类似于库(Index) 里的表，用于对 Document 分类。下面的命令查看 Index 里所有的 Type:
```sh
$curl '127.0.0.1:9200/_mapping?pretty'
``` 

# Document
Document 是实际写入的数据。下面以 Index: users, Type: m 为例演示怎么操作数据。

## 添加数据，修改数据
原型：
```
PUT /{index}/{type}/{id}
```

例子：
```sh
$curl -X PUT '127.0.0.1:9200/users/m/id1' -H'Content-Type:application/json' -d '
{
    "name": "ljj",
    "age": "30"
}
'
```
url 里的“id1”是给定的 id。也可以不给，相应 method 改成 POST，返回的 _id 字段是服务给的 id。

## 删除数据
原型：
```
DELETE /{index}/{type}/{id}
```
例子：
```sh
$curl -X DELETE '127.0.0.0:9200/users/m/id1'
```

# 查询

## 已知 id
原型：
```
GET /{index}/{type}/{id}
```
例子：
```sh
$curl '127.0.0.1:9200/users/m/id1?pretty'
```

## 简易查询
```sh
# 查询
$curl '127.0.0.1:9200/users/_search?q=ljj&pretty'
# 统计
$curl '127.0.0.1:9200/users/_count?q=ljj&pretty'
$curl '127.0.0.1:9200/_search?q=name:ljj&from=1&size=1&_source=name,age'
```
**q 规定了字段，from 规定偏移，size 规定返回条数，_source 规定返回的字段。**

可以看到，Index 和 Type 都是可以去掉的，加上则规定了范围。返回的结果里包含 Index 和 Type。

## 复杂查询
```sh
$curl '127.0.0.1:9200/users/_search' -H'Content-Type:application/json' -d '
{
    "query" : {
        "match": {
            "name": "ljj othername"
        }
    },
    "from": 1,
    "size": 1
}
'
```
**query 规定了匹配规则，match 规定了字段，from 规定偏移，size 规定返回条数。**

name 里面的空格表示 or 的关系。如果需要 and 则使用下面的参数:
```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {"name": "ljj"}
                },
                {
                    "match": {"name": "othername"}
                }
            ]
        }
    }
}
```

## 聚合
聚合相当于 SQL 里的 group by 语句

下面给出一种特别的情况，comment_label 里面是数组，数据类似于
```json
{
    "comment_obj":"58c3f340936edfa7323ec801",
    "eval_star":4,
    "comment_label":[
        "aaa",
        "bbb"
    ]
}
```
使用下面的语句查询
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "term": { //精确匹配，这里用 term 而不是 match
            "comment_obj": "58c3f340936edfa7323ec801"
          }
        },
        {
            "range": {
                "eval_star":{
                    "gte":3
                }
            }
        }
      ]
    }
  },
  "aggs": {//使用 aggs 关键词，也可以用 aggregations
    "comment_label": {
      "terms": {
        "field": "comment_label"
      }
    }
  },
  "size": 0 //size=0 表示只返回统计值，不然还会返回匹配到的文档
}
```
返回的值类似
```json
{
    "took":2,
    "timed_out":false,
    "_shards":{
        "total":5,
        "successful":5,
        "skipped":0,
        "failed":0
    },
    "hits":{
        "total":{
            "value":1,
            "relation":"eq"
        },
        "max_score":null,
        "hits":[

        ]
    },
    "aggregations":{
        "comment_label":{
            "doc_count_error_upper_bound":0,
            "sum_other_doc_count":0,
            "buckets":[
                {
                    "key":"aaa",
                    "doc_count":1
                },
                {
                    "key":"bbb",
                    "doc_count":1
                }
            ]
        }
    }
}
```
已经统计出了所有 comment_obj="58c3f340936edfa7323ec801",eval_star>3 时，所有 comment_label 内值出现次数

现在增加难度，语句嵌套 group by，匹配多个 comment_obj
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {//匹配多个时，使用 terms
            "comment_obj": ["58c3f340936edfa7323ec801"]
          }
        },
        {
            "range": {
                "eval_star":{
                    "gte":3
                }
            }
        }
      ]
    }
  },
  "aggs": {
      "comment_obj":{
          "terms": {
              "field": "comment_obj"
          },
          "aggs":{
            "comment_label": {
              "terms": {
                "field": "comment_label"
              }
            }
          }
      }
  },
  "size": 0
}
```
返回类似
```json
{
    "took":142,
    "timed_out":false,
    "_shards":{
        "total":5,
        "successful":5,
        "skipped":0,
        "failed":0
    },
    "hits":{
        "total":{
            "value":1,
            "relation":"eq"
        },
        "max_score":null,
        "hits":[

        ]
    },
    "aggregations":{
        "comment_obj":{
            "doc_count_error_upper_bound":0,
            "sum_other_doc_count":0,
            "buckets":[
                {
                    "key":"58c3f340936edfa7323ec801",
                    "doc_count":1,
                    "comment_label":{
                        "doc_count_error_upper_bound":0,
                        "sum_other_doc_count":0,
                        "buckets":[
                            {
                                "key":"5f40d73b2fee3a6d338db1ba",
                                "doc_count":1
                            },
                            {
                                "key":"5f40d7452fee3a4e509130fb",
                                "doc_count":1
                            }
                        ]
                    }
                }
            ]
        }
    }
}
```
看到返回值已经按照 comment_obj, comment_label 两极嵌套

# 诊断
## 集群里文档数量
```sh
$curl -XGET '127.0.0.1:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

## 集群健康
```sh
curl -XGET '127.0.0.1:9200/_cluster/health'
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
:-|:-
green | 所有主要分片和复制分片都可用
yallow | 所有主要分片可用，但不是所有复制分片都可用
red | 不是所有主要分片都可用


-------------------------------
参考资料：  
[全文搜索引擎 Elasticsearch 入门教程](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)  
[官方文档](https://www.elastic.co/guide/index.html)  
《Elasticsearch 权威指南》