---
title: 用ab压测
date: 2019-08-13 15:39:10
tags: 运维
id: 1565682061
---
# ab
参数说明：
- -n: 总请求数，请求结束后退出
- -c: 一次产生的请求数，即并发个数
- -p: 模拟post请求，文件格式为gid=2&status=1,配合-T使用
- -T: post数据所使用的Content-Type头信息，比如 -T 'application/x-www-form-urlencoded'

1. 模拟gei请求
```
ab -c 10 -n 10 http://www.test.api.com/?gid=2
```

2. 模拟post请求
在当前目录下创建一个文件post.txt

编辑文件post.txt写入

cid=4&status=1

相当于post传递cid,status参数
```
ab -n 100  -c 10 -p 'post.txt' -T 'application/x-www-form-urlencoded' 'http://test.api.com/ttk/auth/info/'
```

3. 结果分析
```
Server Software:        BWS/1.1
Server Hostname:        www.baidu.com
Server Port:            80

Document Path:          /
Document Length:        154179 bytes

Concurrency Level:      10
Time taken for tests:   0.877 seconds
Complete requests:      10
Failed requests:        9
   (Connect: 0, Receive: 0, Length: 9, Exceptions: 0)
Total transferred:      1549288 bytes
HTML transferred:       1539602 bytes
Requests per second:    11.41 [#/sec] (mean)
Time per request:       876.697 [ms] (mean)
Time per request:       87.670 [ms] (mean, across all concurrent requests)
Transfer rate:          1725.77 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       31   37   5.6     42      42
Processing:   201  571 169.9    540     769
Waiting:       34  107  97.7     50     311
Total:        232  608 171.9    582     811

Percentage of the requests served within a certain time (ms)
  50%    582
  66%    733
  75%    753
  80%    782
  90%    811
  95%    811
  98%    811
  99%    811
 100%    811 (longest request)
```
当结果里 Failed requests 不为0的时候，就表示已经达到系统最高并发了，这时的 Requests per second 较为准确
