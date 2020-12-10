---
title: 用xhprof监控php函数性能和调用链
date: 2020-12-10 15:59:30
tags: php
id: 1607587289
---
# 安装
## 安装扩展
下载安装 xhprof 扩展，在php.ini里添加extension=php_xhprof.co

## 安装web工具
下载地址[https://github.com/facebook/xhprof]
运行http://localhost/xhprof/examples/sample.php 生成测试数据
访问http://localhost/xhprof/xhprof_html/ 查看数据列表
点击[View Full Callgraph] 报错需要安装Graphviz

## 安装Graphviz
从graphviz官网下载 [http://www.graphviz.org/Download.php]
配置环境变量

这是会生成报告
- 红色的矩形部分就是性能开销大，需要优化的函数，
- 白色的矩形部分就是性能开销正常，不需要优化的函数，
- 黄色的矩形部分相对于白色矩形稍微有一些性能开销，但是没有红色矩形那么大，也就是性能开销在白色矩形和红色矩形之间

# XHProf报告字段含义
Function Name：方法名称。
Calls：方法被调用的次数。
Calls%：方法调用次数在同级方法总数调用次数中所占的百分比。
Incl.Wall Time(microsec)：函数运行时间（包括子函数）。（单位：微秒）
IWall%：方法执行花费的时间百分比。
Excl. Wall Time(microsec)：函数运行时间（不包括子函数）。（单位：微秒）
EWall%：方法本身执行花费的时间百分比。
Incl. CPU(microsecs)：函数运行CPU（包括子函数）。（单位：微秒）
ICpu%：方法执行花费的CPU时间百分比。
Excl. CPU(microsec)：函数运行CPU（不包括子函数）。（单位：微秒）
ECPU%：方法本身执行花费的CPU时间百分比。
Incl.MemUse(bytes)：函数运行消耗内存（包括子函数）。（单位：字节）
IMemUse%：方法执行占用的内存百分比。
Excl.MemUse(bytes)：函数运行消耗内存（不包括子函数）。（单位：字节）
EMemUse%：方法本身执行占用的内存百分比。
Incl.PeakMemUse(bytes)：Incl.MemUse峰值。（单位：字节）
IPeakMemUse%：Incl.MemUse峰值百分比。
Excl.PeakMemUse(bytes)：Excl.MemUse峰值。单位：（字节）
EPeakMemUse%：Excl.MemUse峰值百分比。