---
title: SQL练习-学生表
date: 2017/09/23
categories: 学习
tags:
- 学习
- MySQL
---

SQL练习-学生表
================
做面试题做到一道关于学生表的操作，原先有过一些学习，后来又总是断断续续的用起又遗忘，还是要多加练习的。

虽然学生表看着简单，没什么复杂的数据库表关系，但是对SQL的编写的要求却是千变万化，可以有很多变体。

这道面试题的确包含了SQL中不少注意事项，自己也有遗忘，唉。

SQL语句还是要多练习才行。

2017年9月23日更新：增加表的转置。

2017年9月28日更新：完善第一题。

## 基本表和数据

这是一张普通的学生表，包含以下数据：

```sql
mysql> select name,lesson,score from t_stu;

+----+--------+--------+-------+
| id | name   | lesson | score |
+----+--------+--------+-------+
|  1 | 小明   | 语文   |    80 |
|  2 | 小明   | 数学   |    50 |
|  3 | 小明   | 英语   |    90 |
|  4 | 小刚   | 语文   |    70 |
|  5 | 小刚   | 英语   |    85 |
|  6 | 小刚   | 数学   |    90 |
|  7 | 小强   | 语文   |    85 |
|  8 | 小强   | 数学   |    80 |
|  9 | 小强   | 英语   |    70 |
| 10 | 小芳   | 语文   |    60 |
| 11 | 小芳   | 数学   |    50 |
| 12 | 小芳   | 英语   |    70 |
+----+--------+--------+-------+
12 rows in set (0.00 sec)
```

## 各种不同情景

1、找出所有不挂科（不包含60分）的学生的姓名、学科、成绩。

划重点：找出__所有_____不挂科（不包含60分）___的学生的__姓名、学科、成绩__。

需要实现三个功能，两种实现方式：

1） 使用子查询：

```sql
mysql> select name,lesson,score from t_stu where name in (select t.name from (select name,count(*) as jg from t_stu where score>60 group by name having jg=3) as t) order by name,lesson;

+--------+--------+-------+
| name   | lesson | score |
+--------+--------+-------+
| 小刚   | 数学   |    90 |
| 小刚   | 英语   |    85 |
| 小刚   | 语文   |    70 |
| 小强   | 数学   |    80 |
| 小强   | 英语   |    70 |
| 小强   | 语文   |    85 |
+--------+--------+-------+
6 rows in set (0.00 sec)
```

2）使用连接查询（能避免使用in）：

```sql
mysql> select s.name,s.lesson,s.score from t_stu s inner join (select name,count(*) as jg from t_stu where score>60 group by name having jg=3) as t on s.name=t.name order by name,lesson;

+--------+--------+-------+
| name   | lesson | score |
+--------+--------+-------+
| 小刚   | 数学   |    90 |
| 小刚   | 英语   |    85 |
| 小刚   | 语文   |    70 |
| 小强   | 数学   |    80 |
| 小强   | 英语   |    70 |
| 小强   | 语文   |    85 |
+--------+--------+-------+
6 rows in set (0.00 sec)
```

3）更准确的方法：

（2017年9月28日更新）

增加科目，使得每个人的科目数量不一致。

数据表更新后如下：

```sql
mysql> select * from t_stu;

+----+--------+--------+-------+
| id | name   | lesson | score |
+----+--------+--------+-------+
|  1 | 小明   | 语文   |    80 |
|  2 | 小明   | 数学   |    50 |
|  3 | 小明   | 英语   |    90 |
|  4 | 小刚   | 语文   |    70 |
|  5 | 小刚   | 英语   |    85 |
|  6 | 小刚   | 数学   |    90 |
|  7 | 小强   | 语文   |    85 |
|  8 | 小强   | 数学   |    80 |
|  9 | 小强   | 英语   |    70 |
| 10 | 小芳   | 语文   |    60 |
| 11 | 小芳   | 数学   |    50 |
| 12 | 小芳   | 英语   |    70 |
| 13 | 小芳   | 物理   |    80 |
| 14 | 小刚   | 物理   |    50 |
+----+--------+--------+-------+
14 rows in set (0.00 sec)
```

小刚和小芳分别增加了两行新数据，对应新科目“物理”。

首先使用sum和count的两种方法，注意这两种方法的差别。

```sql
mysql> select s.name,count(*) as xk,sum(s.score>60) as jg from t_stu s group by name;

+--------+----+------+
| name   | xk | jg   |
+--------+----+------+
| 小刚   |  4 |    3 |
| 小强   |  3 |    3 |
| 小明   |  3 |    2 |
| 小芳   |  4 |    2 |
+--------+----+------+
4 rows in set (0.00 sec)
```

然后再加上剩余的部分：

```sql
mysql> select s.name,count(*) as xk,sum(s.score>60) as jg from t_stu s group by name having xk=jg;

+--------+----+------+
| name   | xk | jg   |
+--------+----+------+
| 小强   |  3 |    3 |
+--------+----+------+
1 row in set (0.00 sec)
```

这样就可以再完善结合一下，就有以下结果：

```sql
mysql> select s.name,s.lesson,s.score from t_stu s inner join (select name,count(*) as xk,sum(score>60) as jg from t_stu group by name having xk=jg) as t on s.name=t.name order by name,lesson;

+--------+--------+-------+
| name   | lesson | score |
+--------+--------+-------+
| 小强   | 数学   |    80 |
| 小强   | 英语   |    70 |
| 小强   | 语文   |    85 |
+--------+--------+-------+
3 rows in set (0.00 sec)
```

比方法一、二要更完善。

2、找出所有平均分超过80的学生姓名、平均分。

划重点：找出__所有_____平均分超过80___的学生__姓名、平均分__。

要先过滤掉有不及格的同学，因为考虑以下情况：

```java
80*3=240
240-60=180
180/2=90
```

如果是平均分超过90的同学，则无需过滤掉有不及格的同学，因为：

```java
90*3=270
270-60=210
210/2=105
```

将第一题的语句结合起来写，有以下两种实现方式：

1）使用子查询：

```sql
mysql> select t.name,t.pj pinjun from (select name,count(*) as jg,avg(score) as pj from t_stu where score>60 group by name having jg=3 and pj>80) as t order by t.name;

+--------+---------+
| name   | pinjun  |
+--------+---------+
| 小刚   | 81.6667 |
+--------+---------+
1 row in set (0.00 sec)
```

2）使用连接查询：

其实如果不查询具体的信息的话那么连接查询也没什么太大的必要。

3、实现数据表的转置（翻转）

这个确实有难度了，查阅相关资料后总结下两个方法。

1）手动使用连接

先来看几个分开的sql：

i) 找出姓名

```sql
select name from t_stu group by name order by name;
```

ii) 找出某一科的成绩

```sql
select name,score from t_stu where lesson='语文';
```

然后将上面的结合起来，可以有下面的语句：

```sql
mysql> SELECT s.name AS '姓名', yw.score AS '语文', sx.score AS '数学', yy.score AS '英语'
FROM t_stu s LEFT JOIN (SELECT name, score
	FROM t_stu
	WHERE lesson = '语文'
	) yw ON yw.name = s.name LEFT JOIN (SELECT name, score
	FROM t_stu
	WHERE lesson = '数学'
	) sx ON sx.name = s.name LEFT JOIN (SELECT name, score
	FROM t_stu
	WHERE lesson = '英语'
	) yy ON yy.name = s.name
GROUP BY s.name
ORDER BY s.name;

+--------+--------+--------+--------+
| 姓名   | 语文   | 数学   | 英语   |
+--------+--------+--------+--------+
| 小刚   |     70 |     90 |     85 |
| 小强   |     85 |     80 |     70 |
| 小明   |     80 |     50 |     90 |
| 小芳   |     60 |     50 |     70 |
+--------+--------+--------+--------+
4 rows in set (0.00 sec)
```

2) 使用case完成

基于这个[博客](https://www.2cto.com/database/201708/673572.html)

使用的sql语句为：

```sql
mysql> select name '姓名',max(case lesson when '语文' then score end) '语文',max(case lesson when '数学' then score end) '数学',max(case lesson when '英语' then score end) '英语' from t_stu group by name order by name;

+--------+--------+--------+--------+
| 姓名   | 语文   | 数学   | 英语   |
+--------+--------+--------+--------+
| 小刚   |     70 |     90 |     85 |
| 小强   |     85 |     80 |     70 |
| 小明   |     80 |     50 |     90 |
| 小芳   |     60 |     50 |     70 |
+--------+--------+--------+--------+
4 rows in set (0.00 sec)
```

当然还有其他的实现方式，这里就不介绍了。

## 总结

SQL要多加练习，多加练习，多加练习。重要的事情说三遍！！！

写这个题目时，也提醒了我应该乘着大四的剩余时光学习下MySQL的常用优化和SQL优化。加油！
