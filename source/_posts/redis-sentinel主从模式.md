---
title: redis sentinel主从模式
tags: redis
date: 2018-06-01 15:35:26
id: 1528164430
---
# 什么是 sentinel 模式
sentinel 的中文意思是哨兵，即有一个守护进程，时刻检查主服务的状态，如果挂了，就把从服务改成主服务。而客户端都从 sentinel 给的 ip 读写，不用理会服务有没有挂。

一般我们会设置主-从-从，即第二个从服务从第一个从服务同步数据。这样的结构，即保证了主挂掉后还有一个从分担压力，又不会因为一主二从，增加主服务的同步压力。

# 配置服务
## 配置 redis
```sh
$ redis-server --port 30001 --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30001.aof --dbfilename dump-30001.rdb --logfile 30001.log --daemonize yes
$ redis-server --port 30002 --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30002.aof --dbfilename dump-30002.rdb --logfile 30002.log --daemonize yes --slaveof 127.0.0.1:30001
```

## 配置 sentinal
```sh
# Sentinel节点的端口
port 30007
logfile "30007.log"

# 当前Sentinel节点监控 127.0.0.1:30001 这个主节点
# 2代表判断主节点失败至少需要2个Sentinel节点节点同意
# mymaster是主节点的别名
sentinel monitor mymaster 127.0.0.1 30001 2

# 每个Sentinel节点都要定期PING命令来判断Redis数据节点和其余Sentinel节点是否可达，如果超过30000毫秒且没有回复，则判定不可达
sentinel down-after-milliseconds mymaster 30000

# 当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，原来的从节点会向新的主节点发起复制操作，限制每次向新的主节点发起复制操作的从节点个数为1
sentinel parallel-syncs mymaster 1

# 故障转移超时时间为180000毫秒
sentinel failover-timeout mymaster 180000
```

## 启动
有两种方法
```sh
redis-sentinel sentinel-30007.conf
redis-server sentinel-30007.conf
```
改一下端口，其他不变启动 30008, 30009 两台，sentinel 彼此会自动发现对方

## 确认
```sh
$ redis-cli -p 30007 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:30001,slaves=1,sentinels=3 # sentinel=3 表示 sentinel 已经彼此发现
```

现在杀死主
```sh
$ ps aux | grep redis
simon            56248   0.3  0.0  4309624   2028   ??  Ss    8:50下午   0:00.56 ../../src/redis-sentinel *:30008 [sentinel]
simon            56250   0.3  0.0  4309624   2068   ??  Ss    8:50下午   0:00.56 ../../src/redis-sentinel *:30009 [sentinel]
simon            56203   0.2  0.0  4309624   2004   ??  Ss    8:46下午   0:01.04 ../../src/redis-sentinel *:30007 [sentinel]
simon            55341   0.1  0.0  4310648    988   ??  Ss    4:23下午   0:08.04 ../../src/redis-server *:30001
simon            55344   0.1  0.0  4310648   1008   ??  Ss    4:24下午   0:08.10 ../../src/redis-server *:30002

$ kill 55341

$ redis-cli -p 30007 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:30002,slaves=1,sentinels=3
```
此时的 master 地址已经变为 127.0.0.1:30002。

## 查看主从状态
redis 客户端连上 sentinel 后有
```sh
info sentinel
sentinel masters
sentinel slaves mymaster
```
三个常用的命令查看主从状态

# 我踩到的坑
昨天配置了 redis sentinel 的主从切换，无论如何都无法成功，网上也都不到原因。

后来观察 sentinel 是正常工作的，可是服务器之间只传递 sdown 没有传递 odown，于是用这个搜到了。原来 sentinel 有 protected mode，要在配置里添加 bind 0.0.0.0。改完了之后终于切换成功了。

这个东西弄了我有10小时吧，配置模版里提都没提，真是大坑。


-----------------------------
参考资料：  
官方文档 https://redis.io/topics/sentinel  
Redis Sentinel 介绍与部署 https://blog.csdn.net/men_wen/article/details/72724406