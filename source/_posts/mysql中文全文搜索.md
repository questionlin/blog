---
title: mysql中文全文搜索
date: 2019-03-07 10:18:21
tags: mysql
id: 1551925134
---
# 基础
Elasticsearch 是全文搜索的首选，但是对于资金没有那么充足，或者数据量没有那么多的团队，MySQL 提供的全文搜索已经可以满足需求。

MySQL 的全文搜索默认不支持中文，如果要支持中文，要在配置里加入

```
[mysqld]
ngram_token_size=2
```
这个时候 MySQL 可以搜索至少两个字的关键词，1个字还是不能搜索的。然后在建索引的时候要设置 WITH PARSER ngram

```sql
mysql> USE test;

mysql> CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT,
      FULLTEXT (title,body) WITH PARSER ngram
    ) ENGINE=InnoDB CHARACTER SET utf8mb4;

mysql> SET NAMES utf8mb4;

INSERT INTO articles (title,body) VALUES
    ('数据库管理','在本教程中我将向你展示如何管理数据库'),
    ('数据库应用开发','学习开发数据库应用程序');
```

对于已存在的表，这样建立索引
```sql
CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT
     ) ENGINE=InnoDB CHARACTER SET utf8;

ALTER TABLE articles ADD FULLTEXT INDEX ft_index (title,body) WITH PARSER ngram;

# Or:

CREATE FULLTEXT INDEX ft_index ON articles (title,body) WITH PARSER ngram;
```

现在可以这样搜索
```sql
SELECT * FROM articles WHERE MATCH(title,body) AGAINST('数据库教程');
```
注意这里不管是索引还是关键词“数据库教程”，我都没有做分词，MySQL都自动处理了。

# 高级
## IN NATURAL LANGUAGE MODE
默认的搜索方式是 IN NATURAL LANGUAGE MODE，即全部匹配关键词，使用如下
```sql
mysql> SELECT * FROM articles
    WHERE MATCH (title,body)
    AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)
```
即使不加 IN NATURAL LANGUAGE MODE 也是这个结果

## IN BOOLEAN MODE
BOOLEAN MODE 提供高级搜索方法，使用如下
```sql
mysql> SELECT * FROM articles WHERE MATCH (title,body)
    AGAINST ('+MySQL -YourSQL' IN BOOLEAN MODE);
+----+-----------------------+-------------------------------------+
| id | title                 | body                                |
+----+-----------------------+-------------------------------------+
|  1 | MySQL Tutorial        | DBMS stands for DataBase ...        |
|  2 | How To Use MySQL Well | After you went through a ...        |
|  3 | Optimizing MySQL      | In this tutorial we will show ...   |
|  4 | 1001 MySQL Tricks     | 1. Never run mysqld as root. 2. ... |
|  6 | MySQL Security        | When configured properly, MySQL ... |
+----+-----------------------+-------------------------------------+
```
这里关键词里面的
- +表示必须有
- -表示必须没有
- 不加符号表示可有可没有
- @表示距离，即关键词必须在某距离内，例如 MATCH(col1) AGAINST('"word1 word2 word3" @8' IN BOOLEAN MODE)
- < > 这两个符号表示关键词权重，> 提升权重，< 降低权重
- ( ) 表示表达式优先级
- ~ 表示关键词可以没有，如果有，则降低权重
- *匹配以关键词开始的词（见下面例子）
- " 匹配完整的词

例子：
- 'apple banana'  
包含至少一个词

- '+apple + juice'  
必须包含两个词

- '+apple macintosh'  
包含"apple"，如果也包含“macintosh”则提高优先级

- '+apple -macintosh'  
包含'apple'但是不能包含 'macintosh'

- '+apple ~macintosh'  
包含'apple'，如果也包含'macintosh'，则降低优先级

- '+apple +(>turnover <strudel)'  
包含 'apple' 和 'turnover'，或者 'apple' 和 'strudel'（顺序不限），前者比后者优先级高

- 'apple*'  
匹配所有'apple'开头的词，如'apples','applesauce'，或'applet'

- '"some words"'  
匹配完整的词，例如匹配"some words of wisdom"，但不匹配"some noise words"

## with Query Expansion
with Query Expansion 表示联想词。如果用户给的关键词太少，MySQL 会自动联想出其他关键词，例如如果用户搜索'database'，MySQL 会自动匹配 'MySQL', 'Oracle', 'DB2'和'RDBMS'，使用如下

```sql
mysql> SELECT * FROM articles
    WHERE MATCH (title,body)
    AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)

mysql> SELECT * FROM articles
    WHERE MATCH (title,body)
    AGAINST ('database' WITH QUERY EXPANSION);
+----+-----------------------+------------------------------------------+
| id | title                 | body                                     |
+----+-----------------------+------------------------------------------+
|  5 | MySQL vs. YourSQL     | In the following database comparison ... |
|  1 | MySQL Tutorial        | DBMS stands for DataBase ...             |
|  3 | Optimizing MySQL      | In this tutorial we will show ...        |
|  6 | MySQL Security        | When configured properly, MySQL ...      |
|  2 | How To Use MySQL Well | After you went through a ...             |
|  4 | 1001 MySQL Tricks     | 1. Never run mysqld as root. 2. ...      |
+----+-----------------------+------------------------------------------+
6 rows in set (0.00 sec)
```

# 预先分词
ngram 分词是按照词长度来分的，比如当 ngram_token_size=2 时，'分词是按照词长'会被分成'分词 是按 照词 长'这样。效果不是特别好我们更想要自然的分词方法。我的做法是可以先用分词器分好词，比如结巴分词的 cut_for_search，然后用空格分开后保存到专门的列里，这个列的索引使用默认英文模式就行。查询的时候也分好词，这样就能更准确的找到结果了。

可以用我封装的[结巴分词服务](/posts/1540123748)来实现分词接口。不过注意 MyISAM 引擎不用用这种方法，只能用 InnoDB 引擎，不知道为什么。

# 注意
1. 如果索引是 (title,body)，则不能只搜索 title 或 body
2. 有时候我们要去掉表里的某一行，不是删除，而是做一个标记，例如 isDeleted。由于全文索引是单独的索引，不能同时包含 isDeleted 列，所以数据量大的话，建议单独建表来存放，而不是在原表上建立索引

--------------------------------
参考资料：  
MySQL中文支持设置 https://dev.mysql.com/doc/refman/5.7/en/fulltext-search-ngram.html  
BOOLEAN MODE https://dev.mysql.com/doc/refman/5.7/en/fulltext-boolean.html
