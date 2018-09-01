---
title: redis搭建集群
date: 2018-08-30 14:22:44
tags: redis
id: 1535610189
---
# 集群与主从的区别
主从模式中，客户端可以从任何一个服务端读取，分散了读的压力，但是只能对特定的一个服务端做写操作。redis 提供了 sentinel 模式监控主服务的状态，如果主服务挂了，会选择一台从服务作为主服务。可是如果住服务是因为写压力过大，那么相同配置的从服务被选为主之后，毫无疑问也会因为压力太大而挂掉。而且 redis 所有数据都是保存在内存里，如果数据太多，一台服务器放满了，也不能用主从模式。这个时候可以用集群方案来解决。

redis 集群方案里，系统由多个主从服务组成，每个服务端都可以读写，服务端会根据哈希算法，分配数据到特定的服务器上。系统内不再有单点压力。

# 过一遍官方例子
## 在官网下载源码后编译
```sh
$ gem install redis # 安装 ruby 依赖
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

# 手动生成集群
## 生成实例
先来分析一下 create-cluster 的代码。当执行 start 的时候，实际上是执行了
```sh
redis-server --port $PORT --cluster-enabled yes --cluster-config-file nodes-${PORT}.conf --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-${PORT}.aof --dbfilename dump-${PORT}.rdb --logfile ${PORT}.log --daemonize yes
```
$PORT 是30001-30006。如果目录下没有 nodes-${PORT}.conf 配置文件，redis 会自己生成一个，并写入信息。现在先手动生成实例 30001-30004。
```sh
$ ps aux | grep redis
simon            52453   0.1  0.1  4301948   2232   ??  Ss   11:21上午   0:00.08 ../../src/redis-server *:30001 [cluster]
simon            52474   0.1  0.1  4301948   2208   ??  Ss   11:21上午   0:00.02 ../../src/redis-server *:30004 [cluster]
simon            52472   0.1  0.1  4310140   2256   ??  Ss   11:21上午   0:00.03 ../../src/redis-server *:30003 [cluster]
simon            52470   0.1  0.1  4301948   2204   ??  Ss   11:21上午   0:00.04 ../../src/redis-server *:30002 [cluster]
simon            52476   0.0  0.0  4267752    876 s001  S+   11:22上午   0:00.00 grep redis
```
## 连接各节点
只要连接并操作一个一个服务端即可
```sh
127.0.0.1:30001> CLUSTER MEET 127.0.0.1 30002
OK
127.0.0.1:30001> CLUSTER MEET 127.0.0.1 30003
OK
127.0.0.1:30001> CLUSTER MEET 127.0.0.1 30004
OK
```

## 查看集群状态
查看集群状态
```sh
127.0.0.1:30001> cluster info
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:4
cluster_size:0
cluster_current_epoch:3
cluster_my_epoch:1
cluster_stats_messages_ping_sent:213
cluster_stats_messages_pong_sent:213
cluster_stats_messages_meet_sent:3
cluster_stats_messages_sent:429
cluster_stats_messages_ping_received:213
cluster_stats_messages_pong_received:216
cluster_stats_messages_received:429
```

查看集群内所有节点
```sh
127.0.0.1:30001> cluster nodes
735e799f68bf21aa6af679a55f28f69740e2251a 127.0.0.1:30004@40004 master - 0 1535779344092 3 connected
69a812b0dfc02dee94bd03a8e1ff453728409e5a 127.0.0.1:30001@40001 myself,master - 0 1535779343000 1 connected 0-8192
b2897dbc560a9774018559d49eb277ae75bd685b 127.0.0.1:30003@40003 master - 0 1535779343790 0 connected 8193-16383
ac391224d51ccd316809f88e2d59b645213a1e66 127.0.0.1:30002@40002 master - 0 1535779343790 2 connected
```

## 分配哈希槽
redis集群有16384个哈希槽，要把所有数据映射到16384槽，需要批量设置槽
```sh
redis-cli -p 30001 cluster addslots {0..8192}
redis-cli -p 30003 cluster addslots {8193..16383}
```

## 做主从映射
我的配置是 30001主->30002从，30003主->30004从
```sh
127.0.0.1:30002> CLUSTER REPLICATE 69a812b0dfc02dee94bd03a8e1ff453728409e5a
127.0.0.1:30004> CLUSTER REPLICATE b2897dbc560a9774018559d49eb277ae75bd685b
```

## 总结
到此 redis 集群已经搭建好了。现在执行 cluster info，cluster_state 已经是 ok 了。

看得出还是挺麻烦的，推荐使用官方的脚本 redis-trib.rb。当执行 create-cluster create 的时候，是执行了
```sh
../../src/redis-trib.rb create --replicas 1 127.0.0.1:30001 127.0.0.1:3002 127.0.0.1:3003 127.0.0.1:3004 127.0.0.1:3005 127.0.0.1:3006
```
--replicas 1 表示每个主对应一个从。

# 集群更改
## 转移插槽（slot）
```sh
src/redis-trib.rb reshard 127.0.0.1:30001
How many slots do you want to move (from 1 to 16384)? 1000 #转移的插槽数
What is the receiving node ID? b2897dbc560a9774018559d49eb277ae75bd685b #接收的节点
Please enter all the source node IDs. #从指定节点转移还是从其他所有节点转移
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:all
```
以上插槽就搬运好了。如果要把一个节点里的所有插槽转移到另一个节点，可以简单的执行这个命令
```sh
./redis-trib.rb reshard --from <node-id> --to <node-id> --slots <number of slots> --yes <host>:<port>
```

## 集群扩容
首先启动两个新的服务 30005主，30006从。

将30005加入集群
```sh
src/redis-trib.rb add-node 127.0.0.1:30005 127.0.0.1:30001 #引入主节点
src/redis-trib.rb add-node --slave --master-id 03ccad2ba5dd1e062464bc7590400441fafb63f2 127.0.0.1:30006 127.0.0.1:30001 #引入从节点
```
其中 03ccad2ba5dd1e062464bc7590400441fafb63f2 是30005的 cluster node id。然后将其他节点的插槽转移到这个节点就好了。

## 集群缩减
将主节点的插槽全部转移到别的主节点，然后执行
```sh
src/redis-trib.rb del-node 127.0.0.1:30005 03ccad2ba5dd1e062464bc7590400441fafb63f2
```
其中 03ccad2ba5dd1e062464bc7590400441fafb63f2 是节点 cluster node id

# 集群模式的缺陷
1. 键的批量操作支持有限，比如mset, mget，如果多个键映射在不同的槽，就不支持了
2. 键事务支持有限，当多个key分布在不同节点时无法使用事务，同一节点是支持事务
3. 键是数据分区的最小粒度，不能将一个很大的键值对映射到不同的节点
4. 不支持多数据库，只有0，select 0
5. 复制结构只支持单层结构，不支持树型结构。  
6. **集群不能少于3个主节点，否则主从切换会失败**

----------------------------
参考资料：  
官方教程 https://redis.io/topics/cluster-tutorial  
redis集群高可用 https://www.cnblogs.com/leeSmall/p/8414687.html