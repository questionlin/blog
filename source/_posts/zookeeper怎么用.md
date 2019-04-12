---
title: zookeeper怎么用
date: 2019-04-12 17:16:29
tags: 分布式
id: 1555060655
---
这篇文章有点水，不过 zookeeper 在分布式系统中很有分量，所以有必要水一篇。

zookeeper 的功能非常简单，就是一个 k-v 数据库。

下载解压包后，如下操作：

```sh
$ cp conf/zoo_example.cfg conf/zoo.cfg #里面的 tickTime=2000 表示2秒检查一次，分布式锁可以以此作为过期时间
$ bin/zkServer.sh start
$ bin/zkCli.sh -server 127.0.0.1:2181

help #查看帮助
create /foo bar #创建数据
set /foo b #修改数据
create -s /foo bar #得到 /foo0000000001
create -s /foo bar #得到 /foo0000000002
create -e /foo1 bar #得到临时key
```

可以看到用 create -s 时可以用来做递增 id 生成器，create -e 可以用来做分布式锁。

zookeeper功能简单，但是原理非常复杂。是一个每个节点都可读可写的集群，内部用 paxos 算法实现数据同步。因为每个节点都可写，所以性能非常好。
