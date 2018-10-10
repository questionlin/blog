---
title: SQL用join还是分多次查询
date: 2018-10-10 15:52:05
tags: mysql
id: 1539158031
---
这是一个老生常谈的问题了，大家都知道 join 表太多不好，阿里的规范里提到不可以 join 超过3个表。这篇文章通过例子探讨下3个表以内应该用 join 还是多次查询。先说结论，推荐使用 join 语句。

首先打开 mysql 的计时功能
```sql
mysql> set profiling = 1;
mysql> show variables like "%pro%";
+------------------------------------------+-------+
| Variable_name                            | Value |
+------------------------------------------+-------+
| check_proxy_users                        | OFF   |
| have_profiling                           | YES   |
| mysql_native_password_proxy_users        | OFF   |
| performance_schema_max_program_instances | -1    |
| profiling                                | ON    |
| profiling_history_size                   | 15    |
| protocol_version                         | 10    |
| proxy_user                               |       |
| sha256_password_proxy_users              | OFF   |
| slave_compressed_protocol                | OFF   |
| stored_program_cache                     | 256   |
+------------------------------------------+-------+
```
确保 profiling 值为 ON

现在查2000条数据，获得合并查询的时间
```sql
mysql> select * from report_order left join order_form on report_order.order_id = order_form.order_id limit 2000;
```

然后分开查询
```sql
mysql> select * from report_order limit 2000;
mysql> select * from order_form where order_id in (123,123...);
```

然后查看时间
```sql
mysql> show profiles;
+----------+------------+-------------------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                             |
+----------+------------+-------------------------------------------------------------------------------------------------------------------+
|        7 | 0.05155450 | select * from report_order left join order_form on report_order.order_id = order_form.order_id limit 2000         |
|        8 | 0.01129025 | select * from report_order limit 2000                                                                             |
|        9 | 0.08883325 | select * from order_form where order_id in (123,234,345....)                                                      |
|       10 | 0.00051300 | select * from order_form where order_id = '123123'                                                                |
+----------+------------+-------------------------------------------------------------------------------------------------------------------+
```

可以看到，使用 join 的语句7耗时小于多次查询的语句8+9之和，因此在 join 表不多的情况下推荐使用 join。

另外，对于应该使用 in 还是多次查询的问题，可以看到使用 in 的语句9耗时远远小于不使用的语句10*2000，因此推荐使用 in。