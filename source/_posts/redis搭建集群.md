---
title: redis搭建集群
date: 2018-08-30 14:22:44
tags: redis
id: 1535610189
---
# 集群与主从的区别
主从模式中，客户端可以从任何一个服务端读取，分散了读的压力，但是只能对特定的一个服务端做写操作。redis 提供了 sentinel 模式监控主服务的状态，如果主服务挂了，会选择一台从服务作为主服务。可是如果住服务是因为写压力过大，那么相同配置的从服务被选为主之后，毫无疑问也会因为压力太大而挂掉。这个时候可以用集群方案来解决。

redis 集群方案里，系统内不再有单点压力。

# 过一遍官方例子
## 在官网下载源码后编译
```sh
$ wget http://download.redis.io/releases/redis-4.0.11.tar.gz
$ tar xzf redis-4.0.11.tar.gz
$ cd redis-4.0.11
$ make
```
这里确保 src/redis-server, src/redis-cli 两个生成成功。然后进入 utils/create-cluster，执行
```sh
$ ./create-cluster start #生成6个实例
$ ./create-cluster create #将6个实例配置为集群
```

## 用客户端测试集群
官方例子生成了6个实例，分别使用 30001-30006 6个端口。

开一个客户端，连接第一个端口
```sh
$ redis-cli -c -p 30001
127.0.0.1:30001> set ljj 1
OK
```
开另一个客户端，连接第二个端口
```sh
$ redis-cli -c -p 30002
127.0.0.1:30002> get ljj
-> Redirected to slot [1799] located at 127.0.0.1:30001
"1"
127.0.0.1:30001> get ljj
"1"
127.0.0.1:30001> set ljj2 2
OK
```
这里发现客户端已经被转到了30001这个端口。之后用客户端分别连接每个端口，执行 get 和 set 操作，发现 **在任何一个客户端都是可以执行set 和 get 操作的，系统会判断值应该被存储在哪个服务，然后转接过去**。

## 最后清理现场
```sh
$ ./create-cluster stop #停止所有实例
$ ./create-cluster clean #删除所有生成的文件
```

## 检测是否生效


----------------------------
参考资料：  
官方教程 https://redis.io/topics/cluster-tutorial