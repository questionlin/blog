---
title: php进程间通信和共享内存
date: 2018-06-20 10:11:16
tags: php
id: 1529460697
---
## IPC

## 管道和FIFO

## 同步

## 共享内存
mmap, munmap
shm_open, shm_unlink
shmget,shmat,shmdt,shmctl

## php 常驻进程和 socket 通信
php 守护进程需要用到 pcntl_fork() 生成子进程。socket 通信需要用到 socket_ 系列函数。这两个参考资料中的 advanced-php 已经有详细介绍，这篇文章就不写了。也可以看官方文档了解。

--------------------------
参考资料：
《unix网络编程：第二卷》
[advanced-php](https://github.com/elarity/advanced-php)
[PCNTL函数](http://php.net/manual/zh/book.pcntl.php)
[Sockets函数](http://php.net/manual/zh/book.sockets.php)