---
title: sass服务器容量规划与php-fpm配置
date: 2018-08-19 17:38:38
tags: 运维
id: 1534671620
---
对于非 sass 类的应用的容量规划，参考底部《服务器容量规划》。简单的说就是估出总量，然后根据80%的压力落在20%时间的原则，估出QPS，然后给每台机器留30%计算能力，最后得出需要的总机器数。但是对于 sass 应用，商家即使告诉你什么时候会有大活动，他们也估不出会有多少点击量。这个时候就需要另一套配置方法。这套方法是建立在云服务上的，毕竟大部分公司不但不起7x24小时维护。

首先要配置出尽量多的服务器，然后关闭这些服务器。对于很多云服务商来说，这些离线的主机是不收费的。

然后需要一个报警功能，当现存的服务器压力达到预警值的时候或可以自动启动上面关着的服务器，或向运维发出告警，由运维来启动。服务器压力告警只能由本地脚本来完成，因为对外部检测服务来说服务器还是可访问的，对日志收集器来说，服务器可能等不了日志传输和索引的这段时间。

然后就是要评估出服务行业的压力普遍集中的时间，比如出行行业是上下班，周末则要往后延。

下面给一个 php-fpm 配置压力部分的说明（也是一般服务唯一需要配置的部分），详细的说明看底部官方中文文档
```
pm
设置进程管理器如何管理子进程。可用值：static，ondemand，dynamic。必须设置。

static - 子进程的数量是固定的（pm.max_children）。

ondemand - 进程在有需求时才产生（当请求时才启动。与 dynamic 相反，在服务启动时 pm.start_servers 就启动了。

dynamic - 子进程的数量在下面配置的基础上动态设置：pm.max_children，pm.start_servers，pm.min_spare_servers，pm.max_spare_servers。

pm.max_children
pm 设置为 static 时表示创建的子进程的数量，pm 设置为 dynamic 时表示最大可创建的子进程的数量。必须设置。

该选项设置可以同时提供服务的请求数限制。类似 Apache 的 mpm_prefork 中 MaxClients 的设置和 普通PHP FastCGI中的 PHP_FCGI_CHILDREN 环境变量。

pm.start_servers
设置启动时创建的子进程数目。仅在 pm 设置为 dynamic 时使用。默认值：min_spare_servers + (max_spare_servers - min_spare_servers) / 2。

pm.min_spare_servers
设置空闲服务进程的最低数目。仅在 pm 设置为 dynamic 时使用。必须设置。

pm.max_spare_servers
设置空闲服务进程的最大数目。仅在 pm 设置为 dynamic 时使用。必须设置。

pm.max_request
设置每个子进程重生之前服务的请求数。对于可能存在内存泄漏的第三方模块来说是非常有用的。如果设置为 '0' 则一直接受请求，等同于 PHP_FCGI_MAX_REQUESTS 环境变量。默认值：0。

由于php是有内存垃圾回收机制的，所以这个值是可以设的比较大，不过具体还是要看项目。

request_terminate_timeout
设置单个请求的超时中止时间。该选项可能会对 php.ini 设置中的 'max_execution_time' 因为某些特殊原因没有中止运行的脚本有用。设置为 '0' 表示 'Off'。可用单位：s（秒），m（分），h（小时）或者 d（天）。默认单位：s（秒）。默认值：0（关闭）。

```
对于 pm，如果服务器没有弹性扩容方案，也不用给其他程序预留容量的话，选择"static"这个最合适。而"ondemand"最不推荐，因为等到有请求再新建进程，请求会等待较长时间。

---------------------------------
参考资料：  
服务器容量规划：https://ljj.pub/posts/1531898775/  
php-fpm配置说明：http://php.net/manual/zh/install.fpm.configuration.php