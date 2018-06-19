---
title: mysql explain 中各列的意义
date: 2018-06-19 10:22:21
tags: mysql
id: 1529374970
---
### id 列
查询编号，从外到内的顺序递增，临时表的 id 为 NULL。

### select_type 列
这一列显示是哪种类型的 SELECT。最外层是 PRIMARY，其他部分标记如下：
- SUBQUERY
  包含在SELECT列表中的子查询中的SELECT（换句话说，不在FROM子句中）

- DERIVED
  包含在 FROM 子句的子查询中的SELECT。MySQL 会产生一个临时表。

- UNION
  在 UNION 中的第二个和随后的 SELECT。

- UNION RESULT
  用来从UNION的匿名临时表检索结果的SELECT被标记为UNION RESULT

### table 列
这一列显示了对应行正在访问哪个表。

当在FROM子句中有子查询时，table列是<derivedN>的形式，其中N是子查询的id。这总是“向前引用”——换言之，N指向EXPLAIN输出中后面的一行。

当有UNION时，UNION RESULT的table列包含一个参与UNION的id列表。这总是“向后引用”，因为UNION RESULT出现在UNION中所有参与行之后。

### type 列
访问类型，就是 MySQL 决定如何查找表中的行。下面一次从最差到最优：
- ALL
  全表扫描

- index
  索引扫描，也要扫描全表，只是按照索引次序进行而不是行，避免了排序。

- range
  范围查询是一个有限制的索引扫描。WHERE 里带 BETWEEN 或 > <的查询。

- ref
  索引访问。叫 ref 是因为索引要跟某个参考值比较。ref_or_null 是 ref 的一个变体，它意味着 MySQL 必须进行第二次查找以找出 NULL 条目。

- eq_ref
  使用主键或唯一索引

- const, system
  MySQL 对语句优化后，变量被转为常量

- NULL
  只通过索引，不用访问数据表

### possible_keys 列
这一列显示了查询可以使用哪些索引，不一定真的用到。

### key 列
这一列显示了 MySQL 决定采用那个索引。

### key_len 列
该列显示了索引的字节数，不是表中数据的字节数。

### ref 列
这一列显示了之前的表在key列记录的索引中查找值所用的列或常量。

### rows 列
这一列是 MySQL 估计为了找到目标要读取的行数。不是最终目标行数。

### filtered 列
rows 占总行数的比例

### Extra 列
- Using index
  表示 MySQL 使用覆盖索引，不需要访问数据表

- Using where
  WHERE 条件使用了索引

- Using temporary
  使用了临时表

- Using filesort
  使用了外部索引排序

- Range checked for each record (index map: N)
  没有好用的索引


--------------------------
参考资料：《高性能MySQL》