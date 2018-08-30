---
title: elasticsearch搭建集群
date: 2018-08-30 11:36:44
tags: elasticsearch
id: 1535600327
---
# 配置文件
## Elasticsearch 集群中的三种角色

```
1. 此时节点可以成为任何角色
node.master: true #可选为主节点
node.data: true #可存储数据
2. 此时节点从不选举为主节点,只用来存储数据,可作为负载器
node.master: false
node.data: true
3. 此时节点成为主节点,且不存储任何数据,并保有空闲资源,可作为协调器
node.master: true
node.data: false
4. 此时节点既不称为主节点,又不成为数据节点,那么可将他作为搜索器,从节点中获取数据,生成搜索结果等
node.master: false
node.data: false
```

## 本地第一台配置
我的两个服务都在本地，所以要用端口来区分
elasticsearch.yml
```
cluster.name: es-ljj
node.name: node-1
http.port: 9200 #搜索访问的端口
transport.tcp.port: 9300 #节点之间通信的端口
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9301"] #所有节点的地址
```

## 本地第二台配置
elasticsearch.yml
```
cluster.name: es-ljj
node.name: node-2
http.port: 9201 #搜索访问的端口
transport.tcp.port: 9301 #节点之间通信的端口
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9301"] #所有节点的地址
```

## 检测是否生效
这里两个服务都配置了node.master: true，所以先启动的会被选为 master，而后进来的，data 目录下如果已经有其他数据，则会加入失败。

这时我们对两个服务进行操作
```
$curl -X PUT '127.0.0.1:9200/users'
$curl -X PUT '127.0.0.1:9201/pages'
```
然后查看是否已经生效
```
$curl -X GET '127.0.0.1:9200/_cat/indices?v'
$curl -X GET '127.0.0.1:9201/_cat/indices?v'
```
两台返回的都是
```
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   users 5JUXi3KKRtyMRvWvLlwaqg   5   1          0            0      1.2kb           650b
green  open   pages FyR1lz32QYyhJeC-U3_UgQ   5   1          0            0       520b           260b
```
表示集群已经可以使用了

# 动态配置
如果我们要在集群加入一台新服务，或者移除一个老服务，需要修改每个节点配置里的 discovery.zen.ping.unicast.hosts，然后重启每个节点。如果是希望临时修改配置配置，让新的配置马上生效，等到合适的时间再重启每个节点，可以参考 es 提供的[集群配置接口](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)。

# 集群健康
集群健康可以通过 [head 插件](https://github.com/mobz/elasticsearch-head)来监控，也可以使用 es 提供的[集群状态接口](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state.html)。

---------------------------------------
参考资料：  
Elasticsearch5.2.1集群搭建，动态加入节点，并添加监控诊断插件 https://blog.csdn.net/gamer_gyt/article/details/59077189  
集群配置接口 https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html