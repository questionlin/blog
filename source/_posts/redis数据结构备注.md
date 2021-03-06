---
title: redis数据结构备注
date: 2018-07-19 10:37:55
tags: redis
id: 1531967914
---
# Geo
Geo 命令用来把经纬度转换成 [Geohash](https://ljj.pub/posts/1531668376/)。

一般用途：
- 获得地图上一个点附近 x米内的所有点

# Hashes
哈希表。以前会把数据序列化后作为字符串存储，现在可以直接存进 hash 里。缺点是只有一级，所以多级的情况还是要序列化。

一般用途：
- 数据库里的数据原封不动存进 hash

# HyperLogLog
HyperLogLog 的用途是输入大量的字符串，最后得到去重字符串的数量。和 Sets 的区别是速度飞快，而且空间小很多，可能只有千分之一。缺点是数据存入后会做不可逆运算，因此只能统计数量，不能再取出数据。

一般用途：
- 统计 UV

# Lists
列表。以前会用来作为消息队列，现在推荐使用 Streams。

一般用途：
- 有序的队列

# Sets
去重的无序列表。如果要维护一个去重的列表。缺点是无序。

一般用途：
- 分布式的情况下，维护一个去重表

# Sorted Sets
有序的去重列表。同样是去重的列表，可以给每个值加一个权重，得到一个顺序。

一般用途：
- 按照需要的顺序存储数据库的主键。作为数据库索引的补充。

# Streams
专门的消息队列，比 List 快很多，用来代替 Kafka, RabbitMQ 等。

一般用途：
- 消息队列

# Strings
最基本的字符串

一般用途：
- 加锁 SET key value EX 5 NX。SET 和 EX 一起操作，保证锁超时的原子性。
- 存储序列化的数据，比如 json

# Bitmap
图是建立在 String 上的，提供直接对每一位的操作。图的操作直观上就是 offset 上的值是 0 还是 1

一般用途：
- 统计用户一年登录了哪些天 SETBIT $uid $day 1。$offset 是一年中的第几天
- 查看用户是否在线 SETBIT KEY $uid 1。这里 $uid 只能是数字
- 布隆过滤器。旧版可以找插件来实现

# 隐藏结构：Radix
Radix 是 redis 内部用来存储 key 的数据结构，类似 trie 字典树，但是经过压缩（字母合并）。因此可以直接拿 redis 的 key 做字典树来用。

---------------------------
参考资料：  
[HyperLogLog wiki](https://en.wikipedia.org/wiki/HyperLogLog)  
[steams](https://redis.io/topics/streams-intro)