---
title: 线上代码和rpc debug哪家强
tags:
  - php
  - 安利
abbrlink: 45635
date: 2018-06-03 23:16:18
---
标题两种情况，除了看日志外，还有一种做法是做一种转发器，把断点输出转发到程序员这边。这篇文章安利下我做的转发器 [rebugger](https://github.com/questionlin/rebugger)。

rebugger 的原理是用 WorkerMan 做一个中转站，服务器通过 http 请求向中发出消息，中转站提取出内容，转发到程序员这边的 telnet 客户端。例如 PHP 服务器，程序只要执行 file_get_contents() 就能把消息发出去了。

使用 rebugger 的优点是 file_get_contents() 一个函数就把消息发出去了，轻便无依赖。缺点是 get 请求不能有特殊字符，特殊情况需要转义一下。更重的做法是做一个插件，程序引入后可以有一个接口，直接发送 socket 请求给telnet 客户端。不过 rebugger 应该已经能满足大部分 php 程序员的需求了。