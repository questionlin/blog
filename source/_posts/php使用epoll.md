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

先来实现一个纯 PHP 循环向进程发送信号的程序
```php
<?php
// 给当前php进程安装一个alarm信号处理器
// 当进程收到alarm时钟信号后会作出动作
pcntl_signal( SIGALRM, function(){
  echo "tick.".PHP_EOL;
} );
// 定义一个时钟间隔时间，1秒钟吧
$tick = 1;
while( true ){
  // 当过了tick时间后，向进程发送一个alarm信号
  pcntl_alarm( $tick );
  // 分发信号，呼唤起安装好的各种信号处理器
  pcntl_signal_dispatch();
  // 睡个1秒钟，继续
  sleep( $tick );
}
```

代码保存成 timer.php，然后 php timer.php 运行下，如果不出问题应该能跑起来。但是性能不好。下面是一个用 Event 扩展的实现：
```php
<?php
// 初始化一个EventConfig
$eventConfig = new EventConfig();
// 根据EventConfig初始化一个EventBase
$eventBase = new EventBase( $eventConfig );
// 初始化一个定时器event
$timer = new Event( $eventBase, -1, Event::TIMEOUT | Event::PERSIST, function(){
  echo microtime( true )." : 起飞！".PHP_EOL;
} );
// tick间隔为0.05秒钟，我们还可以改成0.5秒钟甚至0.001秒，也就是毫秒级定时器
$tick = 0.05;
// 将定时器event添加（可以不传 $tick）
$timer->add( $tick );
// eventBase进入loop状态
$eventBase->loop();
```

这种定时器是持久的定时器（每隔X时间一定会执行一次），如果想要一次性的定时器（隔X时间后就会执行一次，执行过后再也不执行了），那么将上述代码中的“Event::TIMEOUT | Event::PERSIST”修改为“Event::TIMEOUT”即可。

需要重点说明的是new Event()这行代码了，我把原型贴过来给大家看下：
```
public Event::__construct ( EventBase $base , mixed $fd , int $what , callable $cb [, mixed $arg = NULL ] )
```
- 第一个参数是一个eventBase对象即可
- 第二个参数是文件描述符，可以是一个监听socket、一个连接socket、一个fopen打开的文件或者stream流等。如果是时钟时间，则传入-1。如果是其他信号事件，用相应的信号常量即可，比如SIGHUP、SIGTERM等等
- 第三个参数表示事件类型，依次是Event::READ、Event::WRITE、Event::SIGNAL、Event::TIMEOUT。**其中，加上Event::PERSIST则表示是持久发生，而不是只发生一次就再也没反应了。比如Event::READ | Event::PERSIST就表示某个文件描述第一次可读的时候发生一次，后面如果又可读就绪了那么还会继续发生一次。**
- 第四个参数就熟悉的很了，就是事件回调了，意思就是当某个事件发生后那么应该具体做什么相应
- 第五个参数是自定义数据，这个数据会传递给第四个参数的回调函数，回调函数中可以用这个数据。

如果你有一些自定义用户数据传递给回调函数，可以利用new Event()的第五个参数，这五个参数可以给回调函数用，如下所示：
```php
<?php
$timer = new Event( $eventBase, -1, Event::TIMEOUT | Event::PERSIST, function() use( &$custom ){
  //echo microtime( true )." : 起飞！".PHP_EOL;
  print_r( $custom );
}, $custom = array(
  'name' => 'woshishui',
) );
```

通过以上的案例代码可以总结一下日常流程：
1. 创建EventConfig（非必需）
2. 创建EventBase
3. 创建Event
4. 将Event挂起，也就是执行了Event对象的add方法，不执行add方法那么这个event对象就无法挂起，也就不会执行
5. 将EventBase执行进入循环中，也就是loop方法

以上就是怎么用 Event 扩展来实现一个 epoll 循环程序。可以点下面的参考资料来看验证方法，和如果用 epoll + socket 实现一个 http 服务器。

--------------------------
参考资料：
[advanced-php](https://github.com/elarity/advanced-php/blob/master/13.%20PHP%20socket初探%20---%20硬着头皮继续libevent（二）.md)