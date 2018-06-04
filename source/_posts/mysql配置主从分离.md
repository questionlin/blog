---
title: mysql配置主从分离
tags:
  - mysql
  - 架构
date: 2018-06-03 11:27:39
---
原理：主服务器（Master）负责网站NonQuery操作，从服务器负责Query操作，用户可以根据网站功能模特性块固定访问Slave服务器，或者自己写个池或队列，自由为请求分配从服务器连接。主从服务器利用MySQL的二进制日志文件，实现数据同步。二进制日志由主服务器产生，从服务器响应获取同步数据库。

具体实现：
1. 配置 Master 主服务器
    1. 在 Master MySQL 上创建用户'repl'，并允许其他 Slave 服务可以通过远程访问 Master，通过该用户读取二进制日志，实现数据同步。

    ```
    1 mysql>create user repl; //创建新用户
    2 //repl用户必须具有REPLICATION SLAVE权限，除此之外没有必要添加不必要的权限，密码为mysql。说明一下192.168.0.%，这个配置是指明repl用户所在服务器，这里%是通配符，表示192.168.0.0-192.168.0.255的Server都可以以repl用户登陆主服务器。当然你也可以指定固定Ip。
    3 mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.0.%' IDENTIFIED BY 'mysql';
    ```

    2. 找到MySQL安装文件夹修改my.Ini文件。mysql中有好几种日志方式，这不是今天的重点。我们只要启动二进制日志log-bin就ok。
    在[mysqld]下面增加下面几行代码

    ```
    1 server-id=1   //给数据库服务的唯一标识，一般为大家设置服务器Ip的末尾号
    2 log-bin=master-bin
    3 log-bin-index=master-bin.index
    ```

    3. 查看日志
    ```
    mysql> SHOW MASTER STATUS;
    +-------------------+----------+--------------+------------------+
    | File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
    +-------------------+----------+--------------+------------------+
    | master-bin.000001 | 1285 | | |
    +-------------------+----------+--------------+------------------+
    1 row in set (0.00 sec)
    ```
    重启 MySQL 服务

2. 配置 Slave 从服务器 (windows)
    1. 找到MySQL安装文件夹修改my.ini文件，在[mysqld]下面增加下面几行代码
    ```
    1 [mysqld]
    2 server-id=2
    3 relay-log-index=slave-relay-bin.index
    4 relay-log=slave-relay-bin 
    ```
    重启 MySQL 服务

    2. 连接 Master
    ```
    change master to master_host='192.168.0.104', //Master 服务器Ip
    master_port=3306,
    master_user='repl',
    master_password='mysql', 
    master_log_file='master-bin.000001',//Master服务器产生的日志
    master_log_pos=0;
    ```

    3. 启动 Slave
    ```
    start slave;
    ```

3. Slave 从服务器 (Ubuntu)
    1. 找到 MySQL 安装文件夹修改 my.cnf 文件， vim my.cnf
    ```
    [mysqld]
    basedir =/usr/local/mysql
    datadir =/usr/local/mysql/data
    port = 3306
    server_id = 3
    relay_log_index=slave-relay-bin.index
    relay_log=slave-relay-bin
    ```

    2. ./support-files/myql.server restart 重启MySQL服务  ,  ./bin/mysql 进入MySQL命令窗口 

    3. 连接 Master
    ```
    change master to master_host='192.168.0.104', //Master 服务器Ip
    master_port=3306,
    master_user='repl',
    master_password='mysql', 
    master_log_file='master-bin.000001',//Master服务器产生的日志
    master_log_pos=0;
    ```

    4. 启动 Slave
    ```
    start slave;
    ```

转自：http://www.cnblogs.com/alvin_xp/p/4162249.html