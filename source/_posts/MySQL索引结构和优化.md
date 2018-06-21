---
title: MySQL索引结构和优化
date: 2018-06-09 11:03:52
tags: mysql
id: 1528513470
---
## B-Tree 索引
InnoDB 使用的是B+Tree，即每一个叶子结点都包含指向下一个叶子节点的指针，从而方便叶子节点的范围遍历。B-Tree通常意味着所有的值都是按**顺序**存储在叶子节点的，并且每一个叶子页到根的距离相同。

假设有如下数据表：
```
CREATE TABLE People (
    last_name varchar(50)    not null,
    first_name varchar(50)   not null,
    dob date                 not null,
    gender enum('m', 'f')  not null,
    key(last_name, first_name, dob)
);
```
可以使用B-Tree索引的查询类型。B-Tree索引适用于全键值、键值范围或键前缀查找。**其中键前缀查找只适用于根据最左前缀的查找**。此索引对如下类型的查询有效。

- 全值匹配  
  全值匹配指的是和索引中的所有列进行匹配，例如前面提到的索引可用于查找 last_name 为 Allen, first_name 为 Cuba 、出生于1960-01-01的人。
- 匹配最左前缀  
  索引可用于查找所有 lat_name 为Allen的人，即只使用索引的第一列。
- 匹配列前缀  
也可以只匹配某一列的值的开头部分。例如前面提到的索引可用于查找所有以J开头的姓的人。这里也只使用了索引的第一列。
- 匹配范围值  
例如前面提到的索引可用于查找姓在Allen和Barrymore之间的人。这里也只使用了索引的第一列。
- 精确匹配某一列并范围匹配另外一列  
前面提到的索引也可用于查找所有姓为Allen，并且名字是字母K开头（比如Kim、Karl等）的人。即第一列last_name全匹配，第二列frst_name范围匹配。

### B-Tree索引的限制：
- 如果不是按照索引的最左列开始查找，则无法使用索引。例如上面例子中的索引无法用于查找名字为Bill的人，也无法查找某个特定生日的人，因为这两列都不是最左数据列。类似地，也无法查找姓氏以某个字母结尾的人。
- 不能跳过索引中的列。也就是说，前面所述的索引无法用于查找姓为Smith并且在某个特定日期出生的人。如果不指定名（first_name），则MySQL只能使用索引的第一列。
- 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引优化查找。例如有查询WHERE last_name='Smith' AND frst_name LIKE 'J％' AND dob='1976-12-23'，这个查询只能使用索引的前两列，因为这里LIKE是一个范围条件（但是服务器可以把其余列用于其他目的）。如果范围查询列值的数量有限，那么可以通过使用多个等于条件来代替范围条件。在本章的索引案例学习部分，我们将演示一个详细的案例。

到这里读者应该可以明白，前面提到的索引列的顺序是多么的重要：这些限制都和索引列的顺序有关。在优化性能的时候，可能需要使用相同的列但顺序不同的索引来满足不同类型的查询需求。

## 高性能索引策略
### 独立的列
**索引列不能是表达式的一部分，也不能是函数的参数。** 以下是失败例子：
```
mysql> SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
mysql> SELECT ... WHERE TO_DAYS(CURRENT_DATE) - TO_DAYS(date_col) <= 10;
```

### 前缀索引和索引选择性
有时候需要索引很长的字符列，这会让索引变得大且慢。诀窍在于要选择足够长的前缀以保证较高的选择性，同时又不能太长。  
现在我们有如下数据集
```
mysql>SELECT COUNT(*) AS cnt, city FROM city GROUP BY city ORDER BY cnt DESC LIMIT 10;
-------------------------------------
| cnt | city|
+-----+-----+
| 65 | London |
| 49 | Hiroshima |
| 48 | Teboksary |
| 48 | Pak Kret |
| 48 | Yaound |
| 47 | Tel Aviv-Jaffa |
| 47 | Shimoga |
| 45 | Cabuyao |
| 45 | Callao |
| 45 | Bislig |
```
注意到，上面每个值都出现了45～65次。现在查找到最频繁出现的城市前缀，先从3个前缀字母开始：
```
mysql>SELECT COUNT(*) AS cnt, LEFT(city, 3) AS pref FROM city GROUP BY pref ORDER BY cnt DESC LIMIT 10;
| cnt | pref |
+-----+------+
| 483 | San |
| 195 | Cha |
| 177 | Tan |
| 167 | Sou |
| 163 | al- |
| 163 | Sal |
| 146 | Shi |
| 136 | Hal |
| 130 | Val |
| 129 | Bat |
```
每个前缀都比原来的城市出现的次数更多，因此唯一前缀比唯一城市要少得多。然后我们增加前缀长度，直到这个前缀的选择性接近完整列的选择性。经过实验后发现前缀长度为7时比较合适：
```
mysql>SELECT COUNT(*) AS cnt, LEFT(city, 3) AS pref FROM city GROUP BY pref ORDER BY cnt DESC LIMIT 10;
| cnt | pref |
+-----+------+
| 70 | Santiag |
| 68 | San Fel |
| 65 | London |
| 61 | Valle d |
| 49 | Hiroshi |
| 48 | Teboksa |
| 48 | Pak Kre |
| 48 | Yaound |
| 47 | Tel Avi |
| 47 | Shimoga |
```
计算合适的前缀长度的另外一个办法就是计算完整列的选择性，并使前缀的选择性接近于完整列的选择性。下面显示如何计算完整列的选择性：
```
mysql> SELECT COUNT(DISTINCT LEFT(city, 7))/COUNT(*) FROM city;
| COUNT(DISTINCT city)/COUNT(*) |
+-------------------------------+
| 0.0310 |
```
通常来说（尽管也有例外情况），这个例子中如果前缀的选择性能够接近0.031，基本上就可用了。

在上面的示例中，已经找到了合适的前缀长度，下面演示一下如何创建前缀索引：
```
mysql> ALTER TABLE sakila.city_demo ADD KEY (city(7));
```
前缀索引是一种能使索引更小、更快的有效办法，但另一方面也有其缺点：MySQL无法使用前缀索引做ORDER BY和GROUP BY，也无法使用前缀索引做覆盖扫描。


--------------------------
参考资料：《高性能MySQL》