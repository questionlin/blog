---
title: zookeeper怎么用
date: 2019-04-12 17:16:29
tags: 分布式
id: 1555060655
---
zookeeper 功能简单，但是原理非常复杂，内部用 paxos 算法实现数据同步。

# 安装
```sh
$ cp conf/zoo_example.cfg conf/zoo.cfg #里面的 tickTime=2000 表示2秒检查一次，分布式锁可以以此作为过期时间
$ bin/zkServer.sh start
$ bin/zkCli.sh -server 127.0.0.1:2181

help #查看帮助
```

# 命名服务
```sh
create /foo bar #创建数据
set /foo b #修改数据
create -s /foo bar #得到 /foo0000000001
create -s /foo bar #得到 /foo0000000002
```
可以看到 create -s /foo 的时候实际创建的是 /foo0000000001。foo后面跟的数字是有序的，可以作为集群获得唯一有序名称的来源。

# 注册中心
注册中心的需求是当集群里面有新节点进入或老节点退出的时候，其他节点能得到通知并更新数据。可以使用 zookeeper 的临时数据和观察特定来做到。

create -e 命令创建一个临时数据，当创建者断开连接的时候，数据被清除。利用的是心跳机制。
ls -w 观察一个节点

客户端1
```sh
create /namespace
create -e /namaspace/node1 #得到临时key
```

客户端2
```sh
ls -w /namespace
```

这个时候删除 /namespace/node1 或者直接退出客户端1

客户端2
```sh
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/namespace
```

我们发现当客户端1退出的时候，客户端2收到了通知。这个时候客户端2只要重新从zookeeper拉取一次数据，就得到了最新的数据
[延伸阅读1]

# 分布式锁/集群选主
当一个客户端创建了 /path 数据后，别的客户端就不能创建了。利用这个特性可以实现分布式锁。

如果有一个集群需要一个主节点，这个主节点退出后其他节点得到通知，创建一个临时数据作为锁，申请到的节点就自封为主节点。如果这个主节点退出，那么这个锁会一起被清除。不影响其他节点申请锁。

# zookeeper 选举原理
## paxos 算法
paxos 算法是选举算法的一种，原理是客户端向一个节点写入数据的时候，节点把请求发给其他所有节点，如果得到了半数以上节点同意的回复，返回成功。这里同意的回复表示该节点的数据版本低于更新数据的版本。反对则表示该节点的数据也被另一个客户端改过了，版本号相同或更高。

## fast-paxos 算法
paxos 允许所有节点可写，但是每次写数据都要经过选举，太慢。fast-paxos 算法给系统中选举出一个主节点，只有这个主节点可写，其他节点收到的写请求都被转发到这个主节点。

这种单写的优点是选出主节点后就不需要选举了，而且所有数据是有序的。缺点是单写有性能瓶颈。

## 主节点选举，选举阶段 Leader election
zookeeper 的主节点退出后其他节点会拒绝所有客户端写请求，进入选举状态。每个节点有一个XZID表示本地最新的事务编号
1. 所有节点处于Looking状态，各自依次发起投票，投票包含自己的服务器ID和最新事务ID（ZXID）。
2. 如果发现别人的ZXID比自己大，也就是数据比自己新，那么就重新发起投票，投票给目前已知最大的ZXID所属节点。
3. 每次投票后，服务器都会统计投票数量，判断是否有某个节点得到半数以上的投票。如果存在这样的节点，该节点将会成为准Leader，状态变为Leading。其他节点的状态变为Following。

## 发现阶段 Discovery
1. 为了防止某些意外情况，比如因网络原因在上一阶段产生多个Leader的情况。[延伸阅读2]
2. Leader集思广益，接收所有Follower发来各自的最新epoch值。Leader从中选出最大的epoch，基于此值加1，生成新的epoch分发给各个Follower。
3. 各个Follower收到全新的epoch后，返回ACK给Leader，带上各自最大的ZXID和历史事务日志。Leader选出最大的ZXID，并更新自身历史日志。

## 同步阶段 Synchronization
Leader刚才收集得到的最新历史事务日志，同步给集群中所有的Follower。只有当半数Follower同步成功，这个准Leader才能成为正式的Leader。

---------------------------
延伸阅读
1. 为什么 Apollo 没有使用 zookeeper?

[Apollo](https://github.com/ctripcorp/apollo)使用的是 Eureka。因为 zookeeper 获得通知后要再次订阅，才能再次获得通知。

2. 如果发生脑裂怎么办

脑裂指的是系统彻底分成两个部分，两个部分根据选举都选出了主节点。zookeeper规定服务器必须过半，即数量不到一半的那个网络，会停止服务。

---------------------------
参考：  
https://juejin.im/post/5cdbda12e51d453ccd24650f  
https://blog.csdn.net/u013256816/article/details/80865540