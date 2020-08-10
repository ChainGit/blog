---
title: MySQL复习笔记-I
date: 2018/01/10
categories:
- 学习
tags:
- 学习
- MySQL
---

MySQL复习笔记-I
==============
MySQL中一些重点知识的复习笔记，内容摘自书和一些学习视频，并会持续不断的完善。

测试数据来源：MySQL官方employees数据库。

[MySQL复习笔记-I](https://www.leechain.top/blog/2018/01/10-mysql-review-note-1.html)
[MySQL复习笔记-II](#)
[MySQL复习笔记-III](#)
[MySQL复习笔记-IV](#)

## 基础知识

DB、DBMS、DBA

DDL、DCL、DML

常见MySQL命令行命令。

基础查询、条件查询（where）、常见函数（分组函数）、分组查询（group by/having）、排序查询（order by）、连接查询、子查询、分页查询（limit）、联合查询（union）。

## 基础查询

```sql
/*
基础查询语法：
select 查询列表 from 表名;

特点：
1、查询列表可以是：表中的字段、常量值、表达式、函数
2、查询的结果是一个虚拟的表格
*/

USE myemployees;

#1.查询表中的单个字段
SELECT last_name FROM employees;

#2.查询表中的多个字段
SELECT last_name,salary,email FROM employees;

#3.查询表中的所有字段
## 方式一：
SELECT 
    `employee_id`,
    `first_name`,
    `last_name`,
    `phone_number`,
    `last_name`,
    `job_id`,
    `phone_number`,
    `job_id`,
    `salary`,
    `commission_pct`,
    `manager_id`,
    `department_id`,
    `hiredate` 
FROM
    employees ;
## 方式二：  
 SELECT * FROM employees;

#4.查询常量值
SELECT 100;
SELECT 'john';
 
#5.查询表达式
SELECT 100%98;
 
#6.查询函数 
SELECT VERSION(); 
 
#7.起别名
/*
①便于理解
②如果要查询的字段有重名的情况，使用别名可以区分开来
*/
##方式一：使用as
SELECT 100%98 AS 结果;
SELECT last_name AS 姓,first_name AS 名 FROM employees;

##方式二：使用空格
SELECT last_name 姓,first_name 名 FROM employees;

###案例：查询salary，显示结果为 out put
SELECT salary AS "out put" FROM employees;

#8.去重

##案例：查询员工表中涉及到的所有的部门编号
SELECT DISTINCT department_id FROM employees;

#9.+号的作用
/*
java中的+号：
①运算符，两个操作数都为数值型
②连接符，只要有一个操作数为字符串

mysql中的+号：
仅仅只有一个功能：运算符
若需要连接，使用concat函数。

select 100+90; 两个操作数都为数值型，则做加法运算
select '123'+90;只要其中一方为字符型，试图将字符型数值转换成数值型，如果转换成功，则继续做加法运算
select 'john'+90;	如果转换失败，则将字符型数值转换成0
select null+10; 只要其中一方为null，则结果肯定为null
*/

## 案例：查询员工名和姓连接成一个字段，并显示为“姓名”
SELECT CONCAT('a','b','c') AS 结果;

SELECT 
	CONCAT(last_name,first_name) AS 姓名
FROM
	employees;
```

## 条件查询

```sql
/*
条件查询语法：
	select 
		查询列表
	from
		表名
	where
		筛选条件;

分类：
	一、按条件表达式筛选
	简单条件运算符：> < = != <> >= <=
	
	二、按逻辑表达式筛选
	逻辑运算符：
	  作用：用于连接条件表达式
		  java:  &&   ||    !
		  mysql: and  or   not
		
	&&和and：两个条件都为true，结果为true，反之为false
	||或or： 只要有一个条件为true，结果为true，反之为false
	!或not： 如果连接的条件本身为false，结果为true，反之为false
	
	三、模糊查询
		like
		between and
		in
    not in
		is null
    is not null
    ...
*/
#一、按条件表达式筛选

##案例1：查询工资>12000的员工信息
SELECT 
	*
FROM
	employees
WHERE
	salary>12000;
	
##案例2：查询部门编号不等于90号的员工名和部门编号
SELECT 
	last_name,
	department_id
FROM
	employees
WHERE
	department_id<>90;

#二、按逻辑表达式筛选

##案例1：查询工资z在10000到20000之间的员工名、工资以及奖金
SELECT
	last_name,
	salary,
	commission_pct
FROM
	employees
WHERE
	salary>=10000 AND salary<=20000;
##案例2：查询部门编号不是在90到110之间，或者工资高于15000的员工信息
SELECT
	*
FROM
	employees
WHERE
	NOT(department_id>=90 AND  department_id<=110) OR salary>15000;
#三、模糊查询
/*
like

between and
in
is null|is not null
*/
#1.【like】
/*
特点：
①一般和通配符搭配使用
	通配符：
	% 任意【多】个字符,包含0个字符
	_ 任意【单】个字符
*/
##案例1：查询员工名中包含字符a的员工信息
select 
	*
from
	employees
where
	last_name like '%a%';#abc

##案例2：查询员工名中第三个字符为e，第五个字符为a的员工名和工资
select
	last_name,
	salary
FROM
	employees
WHERE
	last_name LIKE '__n_l%';

##案例3：查询员工名中第二个字符为_的员工名
SELECT
	last_name
FROM
	employees
WHERE
	last_name LIKE '_$_%' ESCAPE '$';

#2.between and
/*
①使用between and 可以提高语句的简洁度
②包含临界值
③两个临界值不要调换顺序
*/
##案例1：查询员工编号在100到120之间的员工信息
SELECT
	*
FROM
	employees
WHERE
	employee_id >= 120 AND employee_id<=100;
##----------------------
SELECT
	*
FROM
	employees
WHERE
	employee_id BETWEEN 120 AND 100;
#3.in
/*
含义：判断某字段的值是否属于in列表中的某一项
特点：
	①使用in提高语句简洁度
	②in列表的值类型必须一致或兼容
	③in列表中不支持通配符
*/
##案例：查询员工的工种编号是 IT_PROG、AD_VP、AD_PRES中的一个员工名和工种编号
SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id = 'IT_PROT' OR job_id = 'AD_VP' OR JOB_ID ='AD_PRES';

##------------------
SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id IN( 'IT_PROT' ,'AD_VP','AD_PRES');

#4、is null
/*
=或<>不能用于判断null值
is null或is not null 可以判断null值
*/

##案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NULL;
##案例2：查询有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NOT NULL;
##----------以下为×
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE 
	salary IS 12000;

#安全等于  <=>
##案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct <=>NULL;

##案例2：查询工资为12000的员工信息
SELECT
	last_name,
	salary
FROM
	employees
WHERE 
	salary <=> 12000;

# 小结
IS NULL: 仅仅可以判断NULL值，可读性较高，建议使用
<=>    : 既可以判断NULL值，又可以判断普通的数值，可读性较低
```

## 常见函数

```sql
/*
常见函数
  概念：类似于java的方法，将一组逻辑语句封装在方法体中，对外暴露方法名
  好处：1、隐藏了实现细节  2、提高代码的重用性
  调用：select 函数名(实参列表) 【from 表】;
特点：
  ①叫什么（函数名）
	②干什么（函数功能）

分类：
	1、单行函数
	如 concat、length、ifnull等
	2、分组函数
	功能：做统计使用，又称为统计函数、聚合函数、组函数
	
常见函数：
	一、单行函数
  
	字符函数：
	length:获取字节个数(utf-8一个汉字代表3个字节,gbk为2个字节)
	concat
	substr
	instr
	trim
	upper
	lower
	lpad
	rpad
	replace
	
	数学函数：
	round
	ceil
	floor
	truncate
	mod
	
	日期函数：
	now
	curdate
	curtime
	year
	month
	monthname
	day
	hour
	minute
	second
	str_to_date
	date_format
  
	其他函数：
	version
	database
	user

	控制函数：
	if（mysql中if是一个函数）
	case
*/

#一、字符函数

#1.length 获取参数值的字节个数
SELECT LENGTH('john');
SELECT LENGTH('张三丰hahaha');

SHOW VARIABLES LIKE '%char%'

#2.concat 拼接字符串
SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;

#3.upper、lower
SELECT UPPER('john');
SELECT LOWER('joHn');
#示例：将姓变大写，名变小写，然后拼接
SELECT CONCAT(UPPER(last_name),LOWER(first_name))  姓名 FROM employees;

#4.substr、substring
注意：索引从1开始
#截取从指定索引处后面所有字符
SELECT SUBSTR('李莫愁爱上了陆展元',7)  out_put;

#截取从指定索引处指定字符长度的字符
SELECT SUBSTR('李莫愁爱上了陆展元',1,3) out_put;

#案例：姓名中首字符大写，其他字符小写然后用_拼接，显示出来
SELECT CONCAT(UPPER(SUBSTR(last_name,1,1)),'_',LOWER(SUBSTR(last_name,2)))  out_put
FROM employees;

#5.instr 返回子串第一次出现的索引，如果找不到返回0
SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷八侠') AS out_put;

#6.trim
SELECT LENGTH(TRIM('    张翠山    ')) AS out_put;
SELECT TRIM('aa' FROM 'aaaaaaaaa张aaaaaaaaaaaa翠山aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa')  AS out_put;

#7.lpad 用指定的字符实现左填充指定长度
SELECT LPAD('殷素素',2,'*') AS out_put;

#8.rpad 用指定的字符实现右填充指定长度
SELECT RPAD('殷素素',12,'ab') AS out_put;

#9.replace 替换
SELECT REPLACE('周芷若周芷若周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;

#二、数学函数

#round 四舍五入
SELECT ROUND(-1.55);
SELECT ROUND(1.567,2);

#ceil 向上取整,返回>=该参数的最小整数
SELECT CEIL(-1.02);

#floor 向下取整，返回<=该参数的最大整数
SELECT FLOOR(-9.99);

#truncate 截断
SELECT TRUNCATE(1.69999,1);

#mod取余
/*
mod(a,b) ：  a-a/b*b

mod(-10,-3):-10- (-10)/(-3)*（-3）=-1
*/
SELECT MOD(10,-3);
SELECT 10%3;

#三、日期函数

#now 返回当前系统日期+时间
SELECT NOW();

#curdate 返回当前系统日期，不包含时间
SELECT CURDATE();

#curtime 返回当前时间，不包含日期
SELECT CURTIME();

#可以获取指定的部分，年、月、日、小时、分钟、秒
SELECT YEAR(NOW()) 年;
SELECT YEAR('1998-1-1') 年;
SELECT  YEAR(hiredate) 年 FROM employees;
SELECT MONTH(NOW()) 月;
SELECT MONTHNAME(NOW()) 月;

#str_to_date 将字符通过指定的格式转换成日期
SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;

#查询入职日期为1992--4-3的员工信息
SELECT * FROM employees WHERE hiredate = '1992-4-3';

SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');

#date_format 将日期转换成字符

SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;

#查询有奖金的员工名和入职日期(xx月/xx日 xx年)
SELECT last_name,DATE_FORMAT(hiredate,'%m月/%d日 %y年') 入职日期
FROM employees
WHERE commission_pct IS NOT NULL;

#四、其他函数
SELECT VERSION();
SELECT DATABASE();
SELECT USER();

#五、流程控制函数

#1.if函数： if else 的效果

SELECT IF(10<5,'大','小');

SELECT last_name,commission_pct,IF(commission_pct IS NULL,'没奖金，呵呵','有奖金，嘻嘻') 备注
FROM employees;

#2.case函数的使用一： switch case 的效果
/*
java中
switch(变量或表达式){
	case 常量1：语句1;break;
	...
	default:语句n;break;
}

mysql中

case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1;
when 常量2 then 要显示的值2或语句2;
...
else 要显示的值n或语句n;
end
*/

/*案例：查询员工的工资，要求

部门号=30，显示的工资为1.1倍
部门号=40，显示的工资为1.2倍
部门号=50，显示的工资为1.3倍
其他部门，显示的工资为原工资
*/

SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;

#3.case 函数的使用二：类似于多重if
/*
java中：
if(条件1){
	语句1；
}else if(条件2){
	语句2；
}
...
else{
	语句n;
}

mysql中：

case 
when 条件1 then 要显示的值1或语句1
when 条件2 then 要显示的值2或语句2
...
else 要显示的值n或语句n
end
*/

#案例：查询员工的工资的情况
如果工资>20000,显示A级别
如果工资>15000,显示B级别
如果工资>10000，显示C级别
否则，显示D级别

SELECT salary,
CASE 
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;
```

## 排序查询

```sql
/*
排序查询语法：
select 查询列表
from 表名
【where  筛选条件】
order by 排序的字段或表达式;

特点：
1、asc代表的是升序，可以省略
  desc代表的是降序
2、order by子句可以支持 单个字段、别名、表达式、函数、多个字段
3、order by子句在查询语句的最后面，除了limit子句
*/

#1、按单个字段排序
SELECT * FROM employees ORDER BY salary DESC;

#2、添加筛选条件再排序

#案例：查询部门编号>=90的员工信息，并按员工编号降序

SELECT *
FROM employees
WHERE department_id>=90
ORDER BY employee_id DESC;

#3、按表达式排序
#案例：查询员工信息 按年薪降序

SELECT *,salary*12*(1+IFNULL(commission_pct,0))
FROM employees
ORDER BY salary*12*(1+IFNULL(commission_pct,0)) DESC;

#4、按别名排序
#案例：查询员工信息 按年薪升序

SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 ASC;

#5、按函数排序
#案例：查询员工名，并且按名字的长度降序

SELECT LENGTH(last_name),last_name 
FROM employees
ORDER BY LENGTH(last_name) DESC;

#6、按多个字段排序

#案例：查询员工信息，要求先按工资降序，再按employee_id升序
SELECT *
FROM employees
ORDER BY salary DESC,employee_id ASC;
```

## 分组查询

```sql
/*
分组查询语法：
  select 查询列表
  from 表
  【where 筛选条件】
  group by 分组的字段
  having 分组后筛选
  【order by 排序的字段】;

特点：
1、和分组函数一同查询的字段必须是group by后出现的字段
2、筛选分为两类：【分组前筛选和分组后筛选】
		针对的表	                		位置		         连接的关键字
分组前筛选（原始表）				      group by前	         where
分组后筛选（group by后的结果集）  group by后	         having

问题1：分组函数做筛选能不能放在where后面
答：不能

问题2：where——group by——having
一般来讲，能用分组前筛选的，尽量使用分组前筛选，提高效率

3、分组可以按单个字段也可以按多个字段
4、可以搭配着排序使用
*/

#引入：查询每个部门的员工个数

SELECT COUNT(*) FROM employees WHERE department_id=90;

#1.简单的分组
##案例1：查询每个工种的员工平均工资
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id;
##案例2：查询每个位置的部门个数
SELECT COUNT(*),location_id
FROM departments
GROUP BY location_id;

#2、可以实现分组前的筛选

##案例1：查询邮箱中包含a字符的每个部门的最高工资
SELECT MAX(salary),department_id
FROM employees
WHERE email LIKE '%a%'
GROUP BY department_id;

##案例2：查询有奖金的每个领导手下员工的平均工资
SELECT AVG(salary),manager_id
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY manager_id;

#3、分组后筛选

##案例1：查询哪个部门的员工个数>5

#①查询每个部门的员工个数
SELECT COUNT(*),department_id
FROM employees
GROUP BY department_id;

#② 筛选刚才①结果
SELECT COUNT(*),department_id
FROM employees
GROUP BY department_id
HAVING COUNT(*)>5;

##案例2：每个工种有奖金的员工的最高工资>12000的工种编号和最高工资
SELECT job_id,MAX(salary)
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY job_id
HAVING MAX(salary)>12000;

##案例3：领导编号>102的每个领导手下的最低工资大于5000的领导编号和最低工资
manager_id>102

SELECT manager_id,MIN(salary)
FROM employees
GROUP BY manager_id
HAVING MIN(salary)>5000;

#4.添加排序

##案例：每个工种有奖金的员工的最高工资>6000的工种编号和最高工资,按最高工资升序
SELECT job_id,MAX(salary) m
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY job_id
HAVING m>6000
ORDER BY m;

#5.按多个字段分组

##案例：查询每个工种每个部门的最低工资,并按最低工资降序
SELECT MIN(salary),job_id,department_id
FROM employees
GROUP BY department_id,job_id
ORDER BY MIN(salary) DESC;
```

## 连接查询

![image](/uploads/0-common-images/17.png)

![image](/uploads/0-common-images/18.jpg)

### SQL92语法

```sql
/*
含义：又称多表查询，当查询的字段来自于多个表时，就会用到连接查询

笛卡尔乘积现象：表1 有m行，表2有n行，结果=m*n行

发生原因：没有有效的连接条件
如何避免：添加有效的连接条件

分类：
	按年代分类：
	    sql92标准：仅仅支持内连接
	    sql99标准【推荐】：支持内连接 + 外连接（左外和右外） + 交叉连接
	
	按功能分类：
		内连接：
			等值连接
			非等值连接
			自连接
		外连接：
			左外连接
			右外连接
			全外连接
		交叉连接
*/
USE employees;

# sql92标准：内连接（仅支持）

## 笛卡尔积（没有有效的连接条件）

SELECT * FROM departments d,employees e;

## 等值连接
 SELECT
  *
FROM
  salaries s,
  employees e
WHERE e.`emp_no` = s.`emp_no`
  AND s.`to_date` = '9999-01-01';

## 不等值连接
 SELECT
  *
FROM
  salaries s,
  employees e
WHERE e.`emp_no` = s.`emp_no`
  AND s.`to_date` = '9999-01-01'
  AND s.`salary` BETWEEN 80000 AND 100000;
  
## 自连接
 SELECT
  *
FROM
  salaries s1,
  salaries s2
WHERE s1.`from_date` = s2.`to_date`;


## 练习：查询工资最高的在职经理所在的部门的工资最低的在职员工的姓名，职位，工资。

/*
嵌套双层查询可以解决 LIMIT & SUBQUERY
*/

### 1.查询出所有的在职经理的员工号

SELECT emp_no FROM dept_manager dm WHERE dm.`to_date`='9999-01-01';

### 2.查出工资最高的经理的员工号
 SELECT
  emp_no,
  salary
FROM
  salaries s
WHERE s.`to_date` = '9999-01-01'
  AND s.emp_no IN
  (SELECT emp_no FROM dept_manager dm WHERE dm.`to_date`='9999-01-01')
ORDER BY s.salary DESC
LIMIT 1;

### 3.查出工资最高的经理的部门号
 SELECT
  dm.dept_no
FROM
  dept_manager dm
WHERE dm.`emp_no` IN
  (SELECT t.emp_no FROM ( SELECT
  emp_no,
  salary
FROM
  salaries s
WHERE s.`to_date` = '9999-01-01'
  AND s.emp_no IN
  (SELECT emp_no FROM dept_manager dm WHERE dm.`to_date`='9999-01-01')
ORDER BY s.salary DESC
LIMIT 1) AS t);
        
### 4.由3结果查询该部门内所有在职员工号
 SELECT
  de.`emp_no`
FROM
  dept_emp de
WHERE de.`dept_no` IN
  ( SELECT
  dm.dept_no
FROM
  dept_manager dm
WHERE dm.`emp_no` IN
  (SELECT t.emp_no FROM (SELECT
  emp_no,
  salary,
  from_date,
  to_date
FROM
  salaries s
WHERE s.`to_date` = '9999-01-01'
  AND s.emp_no IN
  (SELECT
    emp_no
  FROM
    dept_manager dm WHERE dm.`to_date`='9999-01-01')
ORDER BY s.salary DESC
LIMIT 1) AS t)) AND de.`to_date`='9999-01-01';

          
### 5.由4结果查询工资最低的员工号
SELECT s.`emp_no`,s.`salary` FROM salaries s WHERE s.`emp_no` IN (SELECT
  de.`emp_no`
FROM
  dept_emp de
WHERE de.`dept_no` IN
  ( SELECT
  dm.dept_no
FROM
  dept_manager dm
WHERE dm.`emp_no` IN
  (SELECT t.emp_no FROM (SELECT
  emp_no,
  salary,
  from_date,
  to_date
FROM
  salaries s
WHERE s.`to_date` = '9999-01-01'
  AND s.emp_no IN
  (SELECT
    emp_no
  FROM
    dept_manager dm WHERE dm.`to_date`='9999-01-01')
ORDER BY s.salary DESC
LIMIT 1) AS t)) AND de.`to_date`='9999-01-01') AND s.`to_date`='9999-01-01' ORDER BY s.`salary` LIMIT 1;

### 6.由5结果查询出所需信息
 SELECT
  s.`emp_no`,
  CONCAT(
    e.`first_name`,
    ' ',
    e.`last_name`
  ) AS 'name',
  t.`title`,
  s.`salary`
FROM
  employees e,
  titles t,
  (SELECT
    s.`emp_no`,
    s.`salary`
  FROM
    salaries s
  WHERE s.`emp_no` IN
    (SELECT
      de.`emp_no`
    FROM
      dept_emp de
    WHERE de.`dept_no` IN
      (SELECT
        dm.dept_no
      FROM
        dept_manager dm
      WHERE dm.`emp_no` IN
        (SELECT
          t.emp_no
        FROM
          (SELECT
            emp_no,
            salary,
            from_date,
            to_date
          FROM
            salaries s
          WHERE s.`to_date` = '9999-01-01'
            AND s.emp_no IN
            (SELECT
              emp_no
            FROM
              dept_manager dm
            WHERE dm.`to_date` = '9999-01-01')
          ORDER BY s.salary DESC
          LIMIT 1) AS t))
      AND de.`to_date` = '9999-01-01')
    AND s.`to_date` = '9999-01-01'
  ORDER BY s.`salary`
  LIMIT 1) AS s
WHERE e.`emp_no` = t.`emp_no`
  AND t.`to_date` = '9999-01-01'
  AND s.`emp_no` = e.`emp_no`;

+--------+----------------+-------+--------+
| emp_no | name           | title | salary |
+--------+----------------+-------+--------+
|  65337 | Youngkon Munck | Staff |  39821 |
+--------+----------------+-------+--------+
1 row in set (2.60 sec)
```

### SQL99语法

```sql
/*
sql99语法：
	select 查询列表
	from 表1 别名 
	【连接类型】join 
	表2 别名 
	on 连接条件
	【where 筛选条件】
	【group by 分组】
	【having 筛选条件】
	【order by 排序列表】
	
分类：
	内连接（★）：【inner】 join
	外连接
		左外(★):left 【outer】 join
		右外(★)：right 【outer】 join
		全外：full【outer】join
	交叉连接：cross join
*/

USE employees;

SELECT * FROM departments;

SELECT * FROM dept_emp;

# 内连接（相交部分）
/*
语法：
	select 查询列表
	from 表1 别名
	【inner】 join 表2 别名
	on 连接条件;

分类：
	等值
	非等值
	自连接

特点：
	①添加排序、分组、筛选
	②inner可以省略
	③与sql92相比，筛选条件放在where后面，连接条件放在on后面，提高分离性，便于阅读
	④inner join连接和sql92语法中的连接效果是一样的，都是查询多表的【交集】
*/

## 笛卡尔积

/*
cross join是纯粹的笛卡尔积，理论上不应该支持on语法，但是MySQL在这方面做的比较奇怪。
cross join也可以添加on条件。
所以在mysql中，cross join和inner join可以等同，相互替代。
*/

SELECT * FROM employees,departments;

SELECT * FROM employees INNER JOIN departments;

SELECT * FROM employees JOIN departments;

## 等值连接
 SELECT
  *
FROM
  salaries s
  JOIN employees e
    ON e.`emp_no` = s.`emp_no`
    AND s.`to_date` = '9999-01-01'
WHERE e.`first_name` = 'Parto';

# 外连接
 /*
 应用场景：用于查询一个表中有，另一个表没有的记录
 
 特点：
	1、外连接的查询结果为主表中的所有记录
		如果从表中有和它匹配的，则显示匹配的值
		如果从表中没有和它匹配的，则显示null
		【外连接查询结果 = 内连接结果 + 主表中有而从表没有的记录】
	2、左外连接，left join左边的是主表
	   右外连接，right join右边的是主表
	3、左外和右外交换两个表的顺序，可以实现同样的效果 
	4、【全外连接 = 内连接的结果 + 表1中有但表2没有的 + 表2中有但表1没有的】
 */
## 左外连接
 SELECT
  *
FROM
  departments d
  LEFT OUTER JOIN dept_emp de
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01';
    
## 右外连接
 SELECT
  *
FROM
  dept_emp de
  RIGHT OUTER JOIN departments d
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01';
    
## 左外连接（不包含相交部分）
 SELECT
  *
FROM
  departments d
  LEFT OUTER JOIN dept_emp de
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01'
    WHERE de.`dept_no` IS NULL;
    
## 右外连接（不包含相交部分）
 SELECT
  *
FROM
  dept_emp de
  RIGHT OUTER JOIN departments d
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01'
    WHERE de.`dept_no` IS NULL;

## 不包含相交部分
 (SELECT
  *
FROM
  departments d
  LEFT OUTER JOIN dept_emp de
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01'
WHERE de.`dept_no` IS NULL)
UNION
(SELECT
  *
FROM
  departments d
  RIGHT OUTER JOIN dept_emp de
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01'
WHERE d.`dept_no` IS NULL);

## 全连接（包含三部分）
 /*
union默认是去重的，如果不想去重，使用union all
*/
(SELECT
  *
FROM
  departments d
  LEFT OUTER JOIN dept_emp de
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01')
UNION
(SELECT
  *
FROM
  departments d
  RIGHT
  OUTER JOIN dept_emp de
    ON d.`dept_no` = de.`dept_no`
    AND de.`to_date` = '9999-01-01');
    
# 练习：查询工资最高的在职经理所在的部门的工资最低的在职员工的姓名，职位，工资。
 ## 1.查出所有的在职经理
 SELECT
  dm.`emp_no`,
  dm.`dept_no`
FROM
  dept_manager dm WHERE dm.`to_date`='9999-01-01';
  
## 2.查出1中的经理中工资最高的
 SELECT
  s.`emp_no`,
  s.`salary`
FROM
  salaries s
WHERE s.`emp_no` IN
  (SELECT
    dm.`emp_no`
  FROM
    dept_manager dm
  WHERE dm.`to_date` = '9999-01-01')
ORDER BY s.`salary` DESC
LIMIT 1;

## 3.查出2中的经理所在的部门
 SELECT
  dm.`dept_no`
FROM
  dept_manager dm
WHERE dm.`emp_no` IN
  (SELECT
    t.`emp_no`
  FROM
    (SELECT
      s.`emp_no`,
      s.`salary`
    FROM
      salaries s
    WHERE s.`emp_no` IN
      (SELECT
        dm.`emp_no`
      FROM
        dept_manager dm
      WHERE dm.`to_date` = '9999-01-01')
    ORDER BY s.`salary` DESC
    LIMIT 1) AS t);

## 4.查出3中部门的所有在职员工
 SELECT
  de.`emp_no`
FROM
  dept_emp de
WHERE de.`dept_no` IN
  (SELECT
    dm.`dept_no`
  FROM
    dept_manager dm
  WHERE dm.`emp_no` IN
    (SELECT
      t.`emp_no`
    FROM
      (SELECT
        s.`emp_no`,
        s.`salary`
      FROM
        salaries s
      WHERE s.`emp_no` IN
        (SELECT
          dm.`emp_no`
        FROM
          dept_manager dm
        WHERE dm.`to_date` = '9999-01-01')
      ORDER BY s.`salary` DESC
      LIMIT 1) AS t)) AND de.`to_date`='9999-01-01';
      
## 5.查询4中所有员工的工资最低的员工号
 SELECT
  s.`emp_no`,
  s.`salary`
FROM
  salaries s
WHERE s.`emp_no` IN
  (SELECT
    de.`emp_no`
  FROM
    dept_emp de
  WHERE de.`dept_no` IN
    (SELECT
      dm.`dept_no`
    FROM
      dept_manager dm
    WHERE dm.`emp_no` IN
      (SELECT
        t.`emp_no`
      FROM
        (SELECT
          s.`emp_no`,
          s.`salary`
        FROM
          salaries s
        WHERE s.`emp_no` IN
          (SELECT
            dm.`emp_no`
          FROM
            dept_manager dm
          WHERE dm.`to_date` = '9999-01-01')
          AND s.`to_date` = '9999-01-01'
        ORDER BY s.`salary` DESC
        LIMIT 1) AS t))
    AND de.`to_date` = '9999-01-01')
  AND s.`to_date` = '9999-01-01'
ORDER BY s.`salary`
LIMIT 1;

      
## 6.查询5中的员工所需信息
 SELECT
  e.`emp_no`,
  CONCAT(
    e.`first_name`,
    ' ',
    e.`last_name`
  ) AS 'name',
  p.`salary`,
  t.`title`
FROM
  employees e
  JOIN titles t
    ON e.`emp_no` = t.`emp_no`
    AND t.`to_date` = '9999-01-01'
  JOIN
    (SELECT
  s.`emp_no`,
  s.`salary`
FROM
  salaries s
WHERE s.`emp_no` IN
  (SELECT
    de.`emp_no`
  FROM
    dept_emp de
  WHERE de.`dept_no` IN
    (SELECT
      dm.`dept_no`
    FROM
      dept_manager dm
    WHERE dm.`emp_no` IN
      (SELECT
        t.`emp_no`
      FROM
        (SELECT
          s.`emp_no`,
          s.`salary`
        FROM
          salaries s
        WHERE s.`emp_no` IN
          (SELECT
            dm.`emp_no`
          FROM
            dept_manager dm
          WHERE dm.`to_date` = '9999-01-01')
          AND s.`to_date` = '9999-01-01'
        ORDER BY s.`salary` DESC
        LIMIT 1) AS t))
    AND de.`to_date` = '9999-01-01')
  AND s.`to_date` = '9999-01-01'
ORDER BY s.`salary`
LIMIT 1) AS p
    ON p.`emp_no` = e.`emp_no`;

+--------+----------------+--------+-------+
| emp_no | name           | salary | title |
+--------+----------------+--------+-------+
|  65337 | Youngkon Munck |  39821 | Staff |
+--------+----------------+--------+-------+
1 row in set (2.48 sec)
```

## 子查询

```sql
/*
子查询含义：
	出现在其他语句中的select语句，称为子查询或内查询
	外部的查询语句，称为主查询或外查询

分类：
	按子查询出现的位置：
		select后面：
			仅仅支持标量子查询
		from后面：
			支持表子查询
		where或having后面：★
			标量子查询（单行） √
			列子查询  （多行） √
			行子查询
		exists后面（相关子查询）
			表子查询
	
	按结果集的行列数不同：
		标量子查询（结果集只有一行一列）
		列子查询（结果集只有一列多行）
		行子查询（结果集有一行多列）
		表子查询（结果集一般为多行多列）
*/
# where或having后面
 /*
1、标量子查询（单行子查询）
2、列子查询（多行子查询）
3、行子查询（多列多行）

特点：
①子查询放在小括号内
②子查询一般放在条件的右侧
③标量子查询，一般搭配着单行操作符使用
	比如：> < >= <= = <>
列子查询，一般搭配着多行操作符使用
	比如：in、any/some、all
④子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果
*/

## 行子查询（结果集一行多列或多行多列）
### 案例：查询员工编号最小并且工资最高的员工信息
SELECT * 
FROM employees
WHERE (employee_id,salary)=(
	SELECT MIN(employee_id),MAX(salary)
	FROM employees
);

# select后面
 /*
仅仅支持标量子查询
*/
## 案例：查询每个部门的员工个数
 SELECT
  d.*,
  (SELECT
    COUNT(*)
  FROM
    employees e
  WHERE e.department_id = d.`department_id`) 个数
FROM
  departments d;
  
# from后面
 /*
将子查询结果充当一张表，【必须起别名】。
*/
# exists后面（相关子查询）
 /*
语法：
	exists(完整的查询语句)
结果：
	1或0
*/
## 案例：查询有员工的部门名
### in
 SELECT
  department_name
FROM
  departments d
WHERE d.`department_id` IN
  (SELECT
    department_id
  FROM
    employees) 

### exists
   SELECT
    department_name
  FROM
    departments d
  WHERE EXISTS
    (SELECT
      *
    FROM
      employees e
    WHERE d.`department_id` = e.`department_id`);
```

## 分页查询

```sql
/*
【分页查询】
应用场景：
	当要显示的数据，一页显示不全，需要分页提交sql请求
语法：
	select 查询列表
	from 表
	【join type join 表2
	on 连接条件
	where 筛选条件
	group by 分组字段
	having 分组后的筛选
	order by 排序的字段】
	limit 【offset,】size;
	
	offset要显示条目的起始索引（起始索引从0开始）
	size 要显示的条目个数
特点：
	①limit语句放在查询语句的最后
	②公式
		要显示的页数 page，每页的条目数size
	
	select 查询列表
	from 表
	limit (page-1)*size,size;
	
	size=10
	page    
	1	0
	2  	10
	3	20
*/
#案例1：查询前五条员工信息
 SELECT
  *
FROM
  employees
LIMIT 0, 5;

SELECT * FROM  employees LIMIT 5;


#案例2：查询第11条——第25条
SELECT * FROM  employees LIMIT 10,15;


#案例3：有奖金的员工信息，并且工资较高的前10名显示出来
SELECT 
    * 
FROM
    employees 
WHERE commission_pct IS NOT NULL 
ORDER BY salary DESC 
LIMIT 10 ;
```

## 联合查询

```sql
/*
联合查询：
union（联合、合并）：将多条查询语句的结果合并成一个结果

语法：
  (查询语句1)
  union
  (查询语句2)
  union
  ...

应用场景：
要查询的结果来自于多个表，且多个表没有直接的连接关系，但查询的信息一致时

特点：★
1、要求多条查询语句的查询列数是一致的，否则报错。
2、要求多条查询语句的查询的每一列的类型和顺序最好一致，不然数据意义错误。
3、union关键字默认去重，如果使用union all 可以包含重复项。
*/

#引入的案例：查询部门编号>90或邮箱包含a的员工信息
##做法一：
SELECT * FROM employees WHERE email LIKE '%a%' OR department_id>90;
##做法二：
(SELECT * FROM employees  WHERE email LIKE '%a%')
UNION
(SELECT * FROM employees  WHERE department_id>90);

##案例：查询中国用户中男性的信息以及外国用户中年男性的用户信息
(SELECT id,cname FROM t_ca WHERE csex='男')
UNION ALL
(SELECT t_id,tname FROM t_ua WHERE tGender='male';)
```


