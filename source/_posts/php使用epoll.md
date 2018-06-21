---
title: php使用epoll
date: 2018-06-20 16:25:24
tags: php
id: 1529483141
---
之前[这篇](https://ljj.pub/posts/1528164527/)文章介绍了select, poll, epoll 的区别，已经知道 epoll 是最佳选择，也知道了 libevent 是对这三者的封装。这篇文章介绍怎么用 php 和 libevent 来实现一个 epoll 程序。

首先是必要的扩展
1. php 要支持 pcntl
2. 系统已经装好 livevent
3. 安装 php 扩展 event。另一个扩展 libevent 已经很久没更新了。



--------------------------
参考资料：
[advanced-php](https://github.com/elarity/advanced-php)