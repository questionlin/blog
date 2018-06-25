---
title: 分布式事务之mysql两阶段提交
date: 2018-06-25 11:07:58
tags: mysql
id: 1529896095
---
## 两阶段提交
分布式事务即跨数据库事务，要求分布式系统中每个用到的服务器的事务一起提交或一起回滚。为了做到这点，一个解决方法是引入两阶段提交。

两阶段提交简称 2PC，全称 Two Phase Commitment Protocol。要实现两阶段提交需要这两个管理器：
- **资源管理器（resource manager）**：即数据库，用来管理数据，可以实现本地事务。在2PC里扮演参与者角色
- **事务管理器（transaction manager）**：协调每个资源管理器的事务。在2PC里扮演协调者角色

两阶段提交协议分为两个步骤：
1. **准备(prepare)阶段**：即所有的参与者准备执行事务并锁住需要的资源。参与者ready时，向transaction manager报告已准备就绪。 
2. **提交(commit/rollback)阶段**：当transaction manager确认所有参与者都ready后，向所有参与者发送commit命令。 

## MySQL XA事务及其基本语法
MySQL 提供了 XA 事务来支持二段式提交，但它本身不是事务管理器。XA 事务基本语法如下：

XA {START|BEGIN} xid [JOIN|RESUME] 启动xid事务 (xid 必须是一个唯一值; 不支持[JOIN|RESUME]子句) 
XA END xid [SUSPEND [FOR MIGRATE]] 结束xid事务 ( 不支持[SUSPEND [FOR MIGRATE]] 子句) 
XA PREPARE xid 准备、预提交xid事务 
XA COMMIT xid [ONE PHASE] 提交xid事务 
XA ROLLBACK xid 回滚xid事务 
XA RECOVER 查看处于PREPARE 阶段的所有事务

## php 调用 MySQL XA 事务实现分布式事务
首先确保 mysql 开启 XA 事务支持
```sql
SHOW VARIABLES LIKE '%XA%'
```
不是的话执行
```sql
SET innodb_support_xa = ON
```
示例代码如下：
```php
<?php
$dbtest1 = new mysqli("172.20.101.17","public","public","dbtest1")or die("dbtest1 连接失败");
$dbtest2 = new mysqli("172.20.101.18","public","public","dbtest2")or die("dbtest2 连接失败");

//为XA事务指定一个id，xid 必须是一个唯一值。
$xid = uniqid("");

//两个库指定同一个事务id，表明这两个库的操作处于同一事务中
$dbtest1->query("XA START '$xid'");//准备事务1
$dbtest2->query("XA START '$xid'");//准备事务2

try {
    //$dbtest1
    $return = $dbtest1->query("UPDATE member SET name='唐大麦' WHERE id=1") ;
    if($return == false) {
       throw new Exception("库dbtest1@172.20.101.17执行update member操作失败！");
    }

    //$dbtest2
    $return = $dbtest2->query("UPDATE memberpoints SET point=point+10 WHERE memberid=1") ;
    if($return == false) {
       throw new Exception("库dbtest1@172.20.101.18执行update memberpoints操作失败！");
    }

    //阶段1：$dbtest1提交准备就绪
    $dbtest1->query("XA END '$xid'");
    $dbtest1->query("XA PREPARE '$xid'");
    //阶段1：$dbtest2提交准备就绪
    $dbtest2->query("XA END '$xid'");
    $dbtest2->query("XA PREPARE '$xid'");

    //阶段2：提交两个库
    $dbtest1->query("XA COMMIT '$xid'");
    $dbtest2->query("XA COMMIT '$xid'");
} 
catch (Exception $e) {
    //阶段2：回滚
    $dbtest1->query("XA ROLLBACK '$xid'");
    $dbtest2->query("XA ROLLBACK '$xid'");
    die($e->getMessage());
}

$dbtest1->close();
$dbtest2->close();
```

## 两阶段提交的问题
2PC存在同步阻塞、单点问题、脑裂等问题，此外还有数据一致性问题。如果第二阶段参与者和协调者同时挂了，挂了的这个参与者在挂之前已经执行了操作。但是由于他挂了，没有人知道他执行了什么操作。

这种情况下，新的协调者被选出来之后，如果他想负起协调者的责任的话他就只能按照之前那种情况来执行commit或者roolback操作。这样新的协调者和所有没挂掉的参与者就保持了数据的一致性，我们假定他们执行了commit。但是，这个时候，那个挂掉的参与者恢复了怎么办，因为他之前已经执行完了之前的事务，如果他执行的是commit那还好，和其他的机器保持一致了，万一他执行的是roolback操作那？这不就导致数据的不一致性了么？虽然这个时候可以再通过手段让他和协调者通信，再想办法把数据搞成一致的，但是，这段时间内他的数据状态已经是不一致的了！

## 三阶段提交(3PC)及其问题
3PC最关键要解决的就是协调者和参与者同时挂掉的问题，所以3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有CanCommit、PreCommit、DoCommit三个阶段。在第一阶段，只是询问所有参与者是否可可以执行事务操作，并不在本阶段执行事务操作。当协调者收到所有的参与者都返回YES时，在第二阶段才执行事务操作，然后在第三阶段在执行commit或者rollback。

3PC的问题在于，在doCommit阶段，如果参与者无法及时接收到来自协调者的doCommit或者rebort请求时，会在等待超时之后，会继续进行事务的提交。

所以，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

--------------------------
参考资料：
https://blog.csdn.net/soonfly/article/details/70677138
https://blog.csdn.net/yyd19921214/article/details/68953629