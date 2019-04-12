---
title: 实时大数据sql方案
date: 2019-04-10 12:33:07
tags: elasticsearch
id: 1554870854
---

使用SQL的好处是当我们的需求改变的时候，不用改程序，只用改SQL语句即可。但单体sql引擎无法满足大数据的实时计算，这里介绍几种可行的方案。

# 数据库集群
如果系统数据库已经采用了分片集群，那么可以采用分而治之的方案，从各个集群中先统计出部分数据，再归总。

**优点**
- 不用新建系统
- 速度快

**缺点**
- 如果数据库不是集群，则要新建系统
- 无法分而治之的情况不能适用

# ElasticSearch SQL
ES 最近公布了新的 SQL 功能，给开发者带来了福音。三个方案里首选的就是这个，因为搭建方便

**优点**
- 搭建简单

**缺点**


现在没有修改的功能，下面是支持的语句

命令 | 说明
----|-----
DESC table | 用来描述索引的字段属性
SHOW COLUMNS | 功能同上，只是别名
SHOW FUNCTIONS | 列出支持的函数列表，例如count(), sum()，支持通配符过滤
SELECT .. FROM table_name WHERE .. GROUP BY .. HAVING .. ORDER BY .. LIMIT .. | 用来执行查询的命令

现在试试看，先建一条数据
```
POST twitter/doc/
{
  "name":"medcl",
  "twitter":"sql is awesome",
  "date":"2018-07-27",
  "id":123
}
```

然后查询
```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM twitter"
}
```
**因为 SQL 特性是 xpack 的免费功能，所以是在 _xpack 这个路径下面，我们只需要把 SQL 语句传给 query 字段就行了，注意最后面不要加上 ; 结尾，注意是不要！**

下面是比较常用的函数

name      |     type      
----------------|---------------
AVG             | AGGREGATE
COUNT           |AGGREGATE
MAX             |AGGREGATE
MIN             |AGGREGATE
SUM             |AGGREGATE
STDDEV_POP      |AGGREGATE
VAR_POP         |AGGREGATE
PERCENTILE      |AGGREGATE
PERCENTILE_RANK |AGGREGATE
SUM_OF_SQUARES  |AGGREGATE
SKEWNESS        |AGGREGATE
KURTOSIS        |AGGREGATE
DAY_OF_MONTH    |SCALAR   
DAY             |SCALAR   
DOM             |SCALAR   
DAY_OF_WEEK     |SCALAR   
DOW             |SCALAR   
DAY_OF_YEAR     |SCALAR   
DOY             |SCALAR   
HOUR_OF_DAY     |SCALAR   
HOUR            |SCALAR   
MINUTE_OF_DAY   |SCALAR   
MINUTE_OF_HOUR  |SCALAR   
MINUTE          |SCALAR   
SECOND_OF_MINUTE|SCALAR   
SECOND          |SCALAR   
MONTH_OF_YEAR   |SCALAR   
MONTH           |SCALAR   
YEAR            |SCALAR   
WEEK_OF_YEAR    |SCALAR   
WEEK            |SCALAR   
ABS             |SCALAR   
ACOS            |SCALAR   
ASIN            |SCALAR   
ATAN            |SCALAR   
ATAN2           |SCALAR   
CBRT            |SCALAR   
CEIL            |SCALAR   
CEILING         |SCALAR   
COS             |SCALAR   
COSH            |SCALAR   
COT             |SCALAR   
DEGREES         |SCALAR   
E               |SCALAR   
EXP             |SCALAR   
EXPM1           |SCALAR   
FLOOR           |SCALAR   
LOG             |SCALAR   
LOG10           |SCALAR   
MOD             |SCALAR   
PI              |SCALAR   
POWER           |SCALAR   
RADIANS         |SCALAR   
RANDOM          |SCALAR   
RAND            |SCALAR   
ROUND           |SCALAR   
SIGN            |SCALAR   
SIGNUM          |SCALAR   
SIN             |SCALAR   
SINH            |SCALAR   
SQRT            |SCALAR   
TAN             |SCALAR   
SCORE           |SCORE    

# Hadoop + Hive（非实时）
Hadoop 由于是 MapReduce 方案，无法保证实时，但是 Hadoop 生态非常大，可能有实时方案，姑且介绍一下。

Hadoop + Hive 方案是最早的方案，Hadoop 负责存储和分发数据，Hive 负责解析SQL和处理数据。其中 Hive 支持作为 thrift 服务，能够很好的融入到微服务系统中。

**优点**
- 节点数和数据量没有限制
- 在SQL无法满足的情况下可以自己写程序弥补

**缺点**
- 搭建困难
- 不是实时

这里有一份[大数据学习笔记](https://chu888chu888.gitbooks.io/hadoopstudy/) 介绍框架的搭建方案。