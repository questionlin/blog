---
title: mysql集群方案
date: 2019-04-10 15:09:31
tags: mysql
id: 1554880199
---
上一篇文章提到 MySQL 集群方案，除了（[分布式系统的ID](/posts/1553830839)）里提到的通过 ID 分片外，官方有许多集群方案，这篇文章介绍几个。

# InnoDB Cluster (MySQL 7+)
## MySQL Router
之前的文章[mysql配置主从分离](/posts/1528164516) 但是没有说怎么做高可用，MySQL Router 即是扮演这个角色，在 master 不可用的时候，选择一台 slaver 充当主。MySQL Router 原理类似 LVS，有一个 VIP 暴露给应用层，而应用层不需要知道内部数据库服务器的增减。注意 Router 本身不提供高可用，可以通过 keepalived 提供。

配置文件如下：
```sh
[mysql@hdp2~]$more /etc/mysqlrouter.conf
[DEFAULT]
# 日志路径
logging_folder = /home/mysql/mysql-router-2.1.6/log
 
# 插件路径
plugin_folder = /home/mysql/mysql-router-2.1.6/lib/mysqlrouter
 
# 配置路径
config_folder = /home/mysql/mysql-router-2.1.6/config
 
# 运行时状态路径
runtime_folder = /home/mysql/mysql-router-2.1.6/run
 
# 数据文件路径
data_folder = /home/mysql/mysql-router-2.1.6/data
 
[logger]
# 日志级别
level = INFO
 
# 以下选项可用于路由标识的策略部分
[routing:basic_failover]
# Router地址
bind_address = 172.16.1.125
# Router端口
bind_port = 7001
# 读写模式
mode = read-write
# 目标服务器
destinations = 172.16.1.126:3306,172.16.1.127:3306
 
# routing:名称没有要求
[routing:load_balance]
bind_address = 172.16.1.125
bind_port = 7002
mode = read-only
destinations = 172.16.1.126:3306,172.16.1.127:3306
```

配置字段[routing:basic_failover]部分对应写服务器(master)规则，对应SQL: INSERT, UPDATE...，[routing:load_balance]部分对应读服务器(slave)规则，对应SQL: SELECT。如果有其他集群，还可以继续配置。

mode字段可选值有 read-write和read-only

read-write 表示前一个服务器失败后，按照顺序启用后一个服务器。所有的请求都集中在一个服务器。如果前一个服务器恢复了，也不会加回到路由表。只能重启 Router。

read-only 表示一个服务器失败后，会按照顺序启用后一个服务器。所有的请求会按照顺序分配到各个服务器。如果前一个服务器恢复了，可以加回到路由表。

由此可知，写服务只能选择 read-write。如果读服务对一致性有高要求，也应该选择 read-write，如果没有高要求，可以选择 read-only。

查看路由

```sh
C:\WINDOWS\system32>mysql -utest -p123456 -h172.16.1.125 -P7001 -e "show variables like 'server_id'"
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 127   |
+---------------+-------+
```

## 集群配置
- 服务器（至少3台）：svr1, svr2, svr3

配置文件 my.cnf
- 修改不同机器的名称或IP；
- server_id使用不同编号；
- loose-group_replication_group_name使用UUID形式，集群中机器使用同一个UUID；
- loose-group_replication_single_primary_mode在单主模式中为ON，在多主模式中为OFF。

```ini
[mysqld]
# server configuration
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
bind-address    = 0.0.0.0
# Disabling symbolic-links is recommended to prevent assorted security risks
#symbolic-links = 0
# Replication configuration parameters
server_id = 1 #2,3
gtid_mode = ON
enforce_gtid_consistency = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
transaction_write_set_extraction = XXHASH64
binlog_checksum = NONE
log_slave_updates = ON
log_bin = binlog
binlog_format = ROW
relay-log = svr-relay-bin
#
# Group Replication configuration
loose-group_replication_group_name = a38e32fd-5fb6-11e8-ad7a-00259015d941
loose-group_replication_start_on_boot = OFF
loose-group_replication_local_address = svr1:33061
loose-group_replication_group_seeds = svr1:33061,svr2:33061,svr3:3306
loose-group_replication_bootstrap_group = OFF
loose-group_replication_allow_local_disjoint_gtids_join = ON
# Group Replication configuration multi-primary mode
loose-group_replication_single_primary_mode = OFF
loose-group_replication_enforce_update_everywhere_checks = ON
```

创建集群

```sh
$ sudo -i mysqlsh --uri=user@svr1:3306
mysql-js> dba.checkInstanceConfiguration('user@svr1:3306') #检查配置

#创建集群
mysql-js> var cluster = dba.createCluster('mysqlCluster')  # 单主模式集群
mysql-js> var cluster = dba.createCluster('mysqlCluster', {multiMaster:true})  # 多主模式集群
mysql-js> cluster.addInstance('user@svr2:3306')
mysql-js> cluster.addInstance('user@svr3:3306')

#查看状态
mysql-js> var cluster = dba.getCluster()
mysql-js> cluster.status();
```

# NDB Cluster
NDB Cluster 的采用的是 NDB 引擎，和InnoDB最大的区别是隔离级别只支持 Read Committed。因为InnoDB的默认隔离级别是 Repeatable Read，所以在设计数据库和迁移数据库的时候要格外注意这点。NDB Cluster 官方提供了专门的安装引导，这里就不详细说明了。

----------------------------
PS
其实是看到 MySQL Fabric 方案才想写这篇文章的，不过从官方网站上看，好像 Fabric 方案已经被废弃了。

----------------------------
参考资料：  
[适用MySQL Router实现高可用](https://blog.csdn.net/wzy0623/article/details/81103469)  
[MySQL InnoDB Cluster配置](https://www.jianshu.com/p/6e2918845ec8)  
[通过例子理解mysql事务的4种隔离级别](/posts/1528164495)