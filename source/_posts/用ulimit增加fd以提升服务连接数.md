---
title: 用ulimit增加fd以提升服务连接数
date: 2019-02-14 17:09:43
tags: 运维
id: 1550135408
---
之前的文章[《php异步编程》](/posts/1547608999/)中介绍了 php 在常驻系统的情况下，怎么通过异步操作来减少等待时间，增加并发数。但是系统不管是接受客户端的长链接请求（web socket）还是建立上游的链接，都需要打开 file descriptors（文件描述符，简称fd）。系统对 fd 数量是有限制的，增加 fd 数量，就能显著增加并发数。

ulimit 命令可以查看和修改每个用户对系统使用的指标，先来看一下 ulimit 怎么使用

```sh
$ ulimit -a
-t: cpu time (seconds)         unlimited
-f: file size (blocks)         unlimited
-d: data seg size (kbytes)     unlimited
-s: stack size (kbytes)        8192
-c: core file size (blocks)    0
-m: resident set size (kbytes) unlimited
-u: processes                  192276
-n: file descriptors           21000
-l: locked-in-memory size (kb) unlimited
-v: address space (kb)         unlimited
-x: file locks                 unlimited
-i: pending signals            192276
-q: bytes in POSIX msg queues  819200
-e: max nice                   30
-r: max rt priority            65
-N 15:                         unlimited
```

## 推荐的 unlimit 设置
ulimit 分为“硬”和“软”两个指标，“硬”指标限制用户可以开启的进程数，“软”指标限制实际处理运行的进程数（我也不是太清除）。用 -H 和 -S 来查看和修改。下面是 MongoDB 推荐的设置：
- -f (file size): unlimited
- -t (cpu time): unlimited
- -v (virtual memory): unlimited
- -l (locked-in-memory size): unlimited
- -n (open files): 64000
- -m (memory size): unlimited
- -u (processes/threads): 64000

大部分参数都是不限制，只给了我们最关心的 open files 和 processes/threads 限制，因此是比较通用的。linux 默认的 open files 只有1000，加大是非常有必要的。至于64000是不是最合适的值，还需要压测后才能知道。

## 使用 Upstart 的 linux
```sh
limit fsize unlimited unlimited    # (file size)
limit cpu unlimited unlimited      # (cpu time)
limit as unlimited unlimited       # (virtual memory size)
limit memlock unlimited unlimited  # (locked-in-memory size)
limit nofile 64000 64000           # (open files)
limit nproc 64000 64000            # (processes/threads)
```

## 使用 ststemd 的 linux
```sh
[Service]
# Other directives omitted
# (file size)
LimitFSIZE=infinity
# (cpu time)
LimitCPU=infinity
# (virtual memory size)
LimitAS=infinity
# (locked-in-memory size)
LimitMEMLOCK=infinity
# (open files)
LimitNOFILE=64000
# (processes/threads)
LimitNPROC=64000
```

修改后记得重启服务或系统

-----------------------------
参考资料  
https://docs.mongodb.com/manual/reference/ulimit/