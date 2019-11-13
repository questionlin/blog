---
title: mysql运维
date: 2018-06-19 14:17:44
tags:
  - mysql
  - 运维
id: 1529389113
---
# 备份与还原
```sh
# 备份数据库
mysqldump [-h主机名 -P端口] -u用户名 -p密码 数据库名 [表名]> 文件名.sql

# 还原
mysql [-h主机名] -u用户名 -p密码 数据库名 < 文件名.sql
```

# 状态查看
## SHOW STATUS
show status 命令会显示每个服务器变量和值，可以执行以下命令单个查看：
```
show status where Variable_name like 'Conne%'
```
以下是一些重要的：

1. 线程和连接统计
- Connections, Max_used_connections, Threads_connected
- Aborted_clients, Aborted_connects
- Bytes_received, Bytes_sent
- Slow_lanuch_threads, Threads_cached, Threads_created,
Threads_running

如果Aborted_connects不为0，可能意味着网络有问题或某人尝试连接但失败（可能用户指定了错误的密码或无效的数据库，或某个监控系统正在打开TCP的3306端口来检测服务器是否活着）。如果这个值太高，可能有严重的副作用：导致MySQL阻塞一个主机。

2. 二进制日志状态
Binlog_cache_use和Binlog_cache_disk_use状态变量显示了在二进制日志缓存中有多少事务被存储过，以及多少事务因超过二进制日志缓存而必须存储到一个临时文件中。

3. SELECT类型
Select_* 变量是特定类型 SELECT 查询的计数器。其中Select_scan表示全表扫描，Select_range_check 和 Select_full_join 表示无索引的联接。这三个开销较大。

4. 表锁
Table_locks_immediate和Table_locks_waited变量可告诉你有多少锁被立即授权，有多少锁需要等待。但请注意，它们只是展示了服务器级别锁的统计，并不是存储引擎级的锁统计。

## SHOW ENGINE INNODB STATUS
显示 InnoDB 引擎的信息，只有一列。因为不是专业运维，这里只介绍几个需要了解的段。

1. LATEST DETECTED DEADLOCK
只有当前服务器内有死锁时才会出现，内容是死锁的上下文

## SHOW PROCESSLIST
进程列表是当前连接到MySQL的连接或线程的清单。**这个命令可以看哪些线程持有锁**

--------------------------
参考资料：《高性能MySQL》