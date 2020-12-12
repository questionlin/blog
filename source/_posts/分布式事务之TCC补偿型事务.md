---
title: 分布式事务之TCC补偿型事务
date: 2018-07-06 10:14:11
tags: 分布式
id: 1530843273
---
# 原理
之前的一篇[博客](/posts/1529896095/)已经提过通过 MySQL 提供的 XA 事务来实现分布式事务。但很多人发现它性能不好，一些大公司的做法是引入 TCC 事务模型。TCC 事务模型是补偿型事务的一种实现，分为 Try/ Confirm/ Cancel 三个阶段。在解释 TCC 前，现在考虑一个现实中分布式的例子：

假设你要做飞机从上海转机去北京，要求到上海和到北京都有对应时间的机票，只要有一张没有，就两张都不买了。这时候你打电话给航空公司 A，预约保留机票，再打电话给航空公司 B，如果 B 有票的话，再打给 A 确认购票，如果 B 没有票的话，也要打给 A 取消预约。

这个例子里面，向两个公司确认机票对应 Try 阶段，购票对应 Confirm 阶段，取消对应 Cancel 阶段。**因此 TCC 模型就是用业务逻辑实现了数据库的事务逻辑。** 整个逻辑的一个主要方面是确认操作是否可以失败。如果保证成功，则整个操作是一个补偿操作[2]，即隐式确认，明确取消。而 Cancel 阶段就是补偿操作里的补偿部分。一个现实中的例子是，企业通过电话要求客户忽略该信件可以补偿给客户的邮寄信件。

Try/ Confirm/ Cancel 三个阶段对应的数据库操作可以参考下面这个简单的例子：

```sql
# try
INSERT RESERVATION

# confirm
UPDATE RESERVATION SET STATUS='CONFIRMED'

# cancel
UPDATE RESERVATION SET STATUS='CANCELED'

```

# 实现
阿里的实现是增加一个事务管理器，先由主业务发起主事务，然后向每个子业务（远程服务）发起 Try 确认资源。如果资源确认通过的话，再把 Confirm 和 Cancel 的请求代码交给事务管理器，由事务管理器完成后续工作

例如我花20元下单买东西。
## Try阶段
1. 生成订单，状态为UNPAY
2. 支付过程，100-20=80，预扣字段设为20
3. 库存设置为100-1=99，预扣字段设置为1

## Confirm阶段
1. 修改订单状态为PAY
2. 修改用户账号余额，将预扣字段的20块清零
3. 将库存数量的预扣字段设置为0

## Cancel阶段
1. 修改订单状态为CANCEL
2. 把预扣的金额补回到用户的金额里面。80+20=100
3. 将库存预扣部分补回到原本的99+1=100件

# 实现难点
1. 因为完全用业务逻辑实现，每个子事务都要提供三个接口，代码量稍大；
2. 事务要求隔离性（Isolation），即一个事务进行的时候，其他请求读到的应该是原来的数据，这个可以用冗余数据增加版本号来解决；
3. 如果事务管理器挂了，怎么保证事务继续执行？
4. Confirm / Cancel 操作要实现幂等性，即事务管理器失败重启，重新执行的时候，重复上一次的操作不会出错；

--------------------------
参考资料：
1. http://www.enterpriseintegrationpatterns.com/patterns/conversation/TryConfirmCancel.html
2. 补偿操作：http://www.enterpriseintegrationpatterns.com/patterns/conversation/CompensatingAction.html
3. https://cdn.ttgtmedia.com/searchWebServices/downloads/Business_Activities.pdf
4. TCC Java 实现：https://cloud.tencent.com/developer/article/1049345