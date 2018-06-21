---
title: php进程间通信和共享内存
date: 2018-06-20 10:11:16
tags: php
id: 1529460697
---
## 前言
进程间通信简称 IPC，全称 InterProcess Communication。常见的进程间通信方式有：管道（分无名和有名两种）、消息队列、信号量、共享内存和socket。

## 管道和FIFO
管道是最初的 IPC 形式，我们平时使用命令 ps aux | grep php，这里的 | 就是管道。管道最大的局限是没有名字，从而只能由有亲缘关系的进程使用。这一点在 FIFO 出现后得到改进。因而 FIFO 有时也称为命名管道（named pipe）。管道一般是半双工的，但有些系统实现了全双工。

php 使用命名管道通信，创建一个管道的函数叫做posix_mkfifo()，管道创建完成后其实就是一个文件，然后就可以用任何与读写文件相关的函数对其进行操作了，代码大概演示一下：
```php
<?php
// 管道文件绝对路径
$pipe_file = __DIR__.DIRECTORY_SEPARATOR.'test.pipe';
// 如果这个文件存在，那么使用posix_mkfifo()的时候是返回false，否则，成功返回true
if( !file_exists( $pipe_file ) ){
  if( !posix_mkfifo( $pipe_file, 0666 ) ){
    exit( 'create pipe error.'.PHP_EOL );
  }
}
// fork出一个子进程
$pid = pcntl_fork();
if( $pid < 0 ){
  exit( 'fork error'.PHP_EOL );
} else if( 0 == $pid ) {
  // 在子进程中
  // 打开命名管道，并写入一段文本
  $file = fopen( $pipe_file, "w" );
  fwrite( $file, "helo world." );
  exit;
} else if( $pid > 0 ) {
  // 在父进程中
  // 打开命名管道，然后读取文本
  $file = fopen( $pipe_file, "r" );
  // 注意此处fread会被阻塞
  $content = fread( $file, 1024 );
  echo $content.PHP_EOL;
  // 注意此处再次阻塞，等待回收子进程，避免僵尸进程
  pcntl_wait( $status );
}
```
运行结果如下：
```
$ php fifo.php
hello world
```
管道的唯一限制为：
OPEN_MAX  一个进程在任意时刻打开的最大描述符数（Posix 要求至少为16）；  
PIPE_BUF  可原子地写往一个管道或 FIFO 的最大数据量（Posix 要求至少为512）

## 消息队列
这里的消息队列是存储于系统内核中（不是用户态）的一个链表，因而在一个进程发出消息时，不需要另外某个进程等待，这与管道相反。一般我们外部程序使用一个key来对消息队列进行读写操作。在PHP中，是通过msg_*系列函数完成消息队列操作的。
```php
<?php
// 使用ftok创建一个键名，注意这个函数的第二个参数“需要一个字符的字符串”
$key = ftok( __DIR__, 'a' );
// 然后使用msg_get_queue创建一个消息队列
$queue = msg_get_queue( $key, 0666 );
// 使用msg_stat_queue函数可以查看这个消息队列的信息，而使用msg_set_queue函数则可以修改这些信息
//var_dump( msg_stat_queue( $queue ) );  
// fork进程
$pid = pcntl_fork();
if( $pid < 0 ){
  exit( 'fork error'.PHP_EOL );
} else if( $pid > 0 ) {
  // 在父进程中
  // 使用msg_receive()函数获取消息
  msg_receive( $queue, 0, $msgtype, 1024, $message );
  echo $message.PHP_EOL;
  // 用完了记得清理删除消息队列
  msg_remove_queue( $queue );
  pcntl_wait( $status );
} else if( 0 == $pid ) {
  // 在子进程中
  // 向消息队列中写入消息
  // 使用msg_send()向消息队列中写入消息，具体可以参考文档内容
  msg_send( $queue, 1, "hello world" );
  exit;
}
```
运行结果如下：
```
$ php msg.php
hello world
```

## 同步与信号量
为了同步多个进程的活动，就要允许在进程间共享数据。如果要多个进程读写同一个数据，就要引入锁。在多线程的情况下，本身有共享数据缓冲区，上锁与解锁非常简单。但是 php 本身不提供多线程支持，所以不在这里描述。对于多进程上锁与解锁，可以使用信号量(semaphore)。

对于信号量，php 提供 sem_acquire(), sem_get(), sem_release(), sem_remove() 4个函数，已经全部在下个章节，共享内存里面使用到。因为信号量是为了共享内存存在的，就不在这个章节做特别说明了。

## 共享内存
共享内存是最快是进程间通信方式，因为n个进程之间并不需要数据复制，而是直接操控同一份数据。实际上信号量和共享内存是分不开的，要用也是搭配着用。*NIX的一些书籍中甚至不建议新手轻易使用这种进程间通信的方式，因为这是一种极易产生死锁的解决方案。共享内存顾名思义，就是一坨内存中的区域，可以让多个进程进行读写。这里最大的问题就在于数据同步的问题，比如一个在更改数据的时候，另一个进程不可以读，不然就会产生问题。所以为了解决这个问题才引入了信号量，信号量是一个计数器，是配合共享内存使用的，一般情况下流程如下：
- 当前进程获取将使用的共享内存的信号量
- 如果信号量大于0，那么就表示这块儿共享资源可以使用，然后进程将信号量减1
- 如果信号量为0，则进程进入休眠状态一直到信号量大于0，进程唤醒开始从1

一个进程不再使用当前共享资源情况下，就会将信号量减1。这个地方，信号量的检测并且减1是原子性的，也就说两个操作必须一起成功，这是由系统内核来实现的。

```php
<?php
// sem key
$sem_key = ftok( __FILE__, 'b' );
$sem_id = sem_get( $sem_key );
// shm key
$shm_key = ftok( __FILE__, 'm' );
$shm_id = shm_attach( $shm_key, 1024, 0666 );
const SHM_VAR = 1;
$child_pid = [];
// fork 2 child process
for( $i = 1; $i <= 2; $i++ ){
  $pid = pcntl_fork();
  if( $pid < 0 ){
    exit();
  } else if( 0 == $pid ) {
	// 获取锁
	sem_acquire( $sem_id );
	if( shm_has_var( $shm_id, SHM_VAR ) ){
	  $counter = shm_get_var( $shm_id, SHM_VAR );
	  $counter += 1;
	  shm_put_var( $shm_id, SHM_VAR, $counter );
	} else {
	  $counter = 1;
	  shm_put_var( $shm_id, SHM_VAR, $counter );
	}
	// 释放锁，一定要记得释放，不然就一直会被阻锁死
  sem_release( $sem_id );
  // 释放后删除
  sem_remove( $sem_id );
	exit;
  } else if( $pid > 0 ) {
    $child_pid[] = $pid;
  }
}
while( !empty( $child_pid ) ){
  foreach( $child_pid as $pid_key => $pid_item ){
    pcntl_waitpid( $pid_item, $status, WNOHANG );
	unset( $child_pid[ $pid_key ] );
  }
}
// 休眠2秒钟，2个子进程都执行完毕了
sleep( 2 );
echo '最终结果'.shm_get_var( $shm_id, SHM_VAR ).PHP_EOL;
// 记得删除共享内存数据，删除共享内存是有顺序的，先remove后detach，顺序反过来php可能会报错
shm_remove( $shm_id );
shm_detach( $shm_id );
```
运行结果如下：
```
$ php shm.php
最终结果2
```

确切说，如果不用sem的话，上述的运行结果在一定概率下就会产生1而不是2。但是只要加入sem，那就一定保证100%是2，绝对不会出现其他数值。

## php 守护进程和 socket 通信
进程间通信的前提是 php 需要是守护进程，不然还没收到信息就退出了。php 守护进程需要用到 pcntl_fork() 生成子进程。socket 通信需要用到 socket_ 系列函数。这两个参考资料中的 advanced-php 已经有详细介绍，这篇文章就不写了。也可以看官方文档了解。

--------------------------
参考资料：
《unix网络编程：第二卷》
[advanced-php](https://github.com/elarity/advanced-php)
[PCNTL函数](http://php.net/manual/zh/book.pcntl.php)
[Sockets函数](http://php.net/manual/zh/book.sockets.php)
[posix函数](http://php.net/manual/zh/ref.posix.php)
[Semaphore函数](http://php.net/manual/zh/book.sem.php)