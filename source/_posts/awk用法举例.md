---
title: awk用法举例
date: 2019-07-19 10:16:46
tags: 常用命令
id: 1563502663
---
awk 是用来读取文件的每一行，每一列（默认空格分开），然后打印出来的程序。基本语法是
```
awk '正则表达式 {操作}' 文件名
```
# 基本操作
下面是一些例子。先来生成一个文件a.txt，内容是
```
11 12 13 14 15
21 22 23 24 25
31 32 33 34 35
```

## 初次尝试
print 命令表示打印，这个表达式会打印整个文件
```
$ awk '{ print }' a.txt
11 12 13 14 15
21 22 23 24 25
31 32 33 34 35
```

在 print 后面跟 $0 表示列数，$0表示整行（不分列），$1 表示第一列，这个表达式也是打印整个文件
```
$ awk '{ print $0 }' a.txt
11 12 13 14 15
21 22 23 24 25
31 32 33 34 35
```

"" 里面放的是要打印字符串，这个命令针对每一行打印 ljj
```
$ awk '{ print "ljj" }' a.txt
ljj
ljj
ljj
```

## 多个值
-F后面跟分隔列的字符串，a.txt是用空格分隔，所以这里跟" "。这里打印第一列和第三列
```
$ awk -F" " '{ print $1 $3 }' a.txt
1113
2123
3133
```

上面这个把第一列和第三列连起来了，在中间加 " " 隔开
```
$ awk -F" " '{ print $1 " " $3 }' a.txt
11 13
21 23
31 33
```

添加你要的任意字符串
```
$ awk -F" " '{ print "第一列：" $1 " 第三列：" $3 }' a.txt
第一列：11 第三列：13
第一列：21 第三列：23
第一列：31 第三列：33
```

## 扩展脚本
如果你的命令很复杂，还要多次使用，那么可以保存成文件，新建一个文件 myscript.awk
```
BEGIN {
    FS=" "
}
{ print $1 }
```
然后运行
```
$ awk -f myscript.awk a.txt
```

## 正则表达式
先来个简单的，打印有 foo 的行
```
$ awk '/foo/ {print}'
```

## 等式
打印第一列等于 11 的行的第三列
```
$ awk '$1 == "11" {print $3}' a.txt
13
```
这种等式还支持 “==”, “<“, “>”, “<=”, “>=”, 和 “!=”。此外，“~” 和 “!~” 后面可以跟正则表达式，分别表示包含和不包含
```
$ awk '$1 ~ /11/ {print $3}' a.txt
```

## 条件
类似程序代码，awk 也支持条件，比如
```
{ 
  if ( $5 ~ /root/ ) { 
          print $3 
  } 
}
```
或者更复杂一点
```
{ 
  if ( $1 == "foo" ) { 
           if ( $2 == "foo" ) { 
                    print "uno" 
           } else { 
                    print "one" 
           } 
  } else if ($1 == "bar" ) { 
           print "two" 
  } else { 
           print "three" 
  } 
}
```

表达式还能做逻辑判断
```
( $1 == "foo" ) && ( $2 == "bar" ) { print }
```

## 分隔符
上面用 -F" " 表示用空格座位分隔符，其实 -F 也支持正则表达式。比如多个制表符
```
-F"\t+"
```
或者别的
```
-F"foo[0‑9][0‑9][0‑9]"
```

## 列数
NF 表示列数（Number of fields），下面判断列数
```
$ awk 'NF == 5 {print}' a.txt
11 12 13 14 15
21 22 23 24 25
31 32 33 34 35
```
5改成别的数字就不输出了

## 行数
NR 表示行数(Number of record)，下面判断行数，排除了第2行
```
$ awk '(NR<2) || (NR>2) {print}' a.txt
11 12 13 14 15
31 32 33 34 35
```

# 实现SQL
先新建两个文件
user，字段 id name addr
```
1 zhangsan hubei
3 lisi tianjin
4 wangmazi guangzhou
2 wangwu beijing
```

consumer，字段 id cost date
```
1 15 20121213
2 20 20121213
3 100 20121213
4 99 20121213
1 25 20121114
2 108 20121114
3 100 20121114
4 66 20121114
1 15 20121213
1 115 20121114
```

## where 条件过滤
```
select * from user; 
awk 1 user;
select * from consumer where cost > 100;
awk '$2>100' consumer
```

## 去重 distinct
```
select distinct(date) from consumer;
awk '!a[$3]++ {print $3}' consumer
select distinct(*) from consumer;
awk '!a[$0]++' consumer
```
这里新建了一个变量数组 a[]，当 a 里没有 key 的时候打印，即第一次打印，余下都跳过

## 排序 order by
```
select id from user order by id;
awk '{a[$1]}END{asorti(a);for(i=1;i<=length(a);i++) {print a[i]}}' user
```

## limit
```
select * from consumer limit 2;
awk 'NR<=2' consumer
awk 'NR>2{exit}1' consumer # performance is better
```

## 分组求和统计，关键词：group by、having、sum、count
```
select id, count(1), sum(cost) from consumer group by id having count(1) > 2;
awk '{a[$1]=a[$1]==""?$2:a[$1]","$2}END{for(i in a){c=split(a[i],b,",");if(c>2){sum=0;for(j in b){sum+=b[j]};print i"\t"c"\t"sum}}}' consumer
```

## 模糊查询，关键词：like（like属于通配，也可正则 REGEXP）
```
select name from user where name like 'wang%';
awk '$2 ~/^wang/{print $2}' user
select addr from user where addr like '%bei';
awk '/.*bei$/{print $3}' user
select addr from user where addr like '%bei%';
awk '$3 ~/bei/{print $3}' user
```

## 多表 join 关联查询，关键词：join
```
select a.* , b.* from user a inner join consumer b  on a.id = b.id and b.id = 2;
awk 'ARGIND==1{a[$1]=$0;next}{if(($1 in a)&&$1==2){print a[$1]"\t"$2"\t"$3}}' user consumer
```

## 多表水平联接，关键词：union all
```
select a.* from user a union all select b.* from user b;
awk 1 user user
select a.* from user a union select b.* from user b;
awk '!a[$0]++' user user
```

## 随机抽样统计，关键词：order by rand()
```
SELECT * FROM consumer ORDER BY RAND() LIMIT 2;
awk 'BEGIN{srand();while(i<2){k=int(rand()*10)+1;if(!(k in a)){a[k];i++}}}(NR in a)' consumer
```

------------------------------
参考资料：  
https://developer.ibm.com/tutorials/l-awk1/  
https://my.oschina.net/leejun2005/blog/100710#OSC_h3_5