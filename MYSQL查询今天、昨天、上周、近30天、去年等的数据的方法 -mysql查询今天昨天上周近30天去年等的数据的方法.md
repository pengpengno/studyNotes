---
title: MYSQL查询今天、昨天、上周、近30天、去年等的数据的方法 
date: 2021-11-23 13:56:44.244
updated: 2022-04-27 16:35:47.946
url: /archives/mysql查询今天昨天上周近30天去年等的数据的方法
categories: 
- 数据库
tags: 
---



## 一、MySQL查询今天、昨天、上周、近30天、去年等的数据的方法：
[TOC]
 

##### 1.1 删除建立时间超过3天的订单记录
 
```
delete 订单表 where datediff( dd, order_addtime, getdate() ) > 3 用函数datediff() datediff( dd, 时间1, 时间2 )，意思是：计算时间1到时间2之间的天数 所以，datediff( dd, order_addtime, getdate() ) > 3，就是超过3天的
```
##### 1.2 今天  
```
select * from 表名 where to_days(时间字段名) = to_days(now());  
```
##### 1.3 昨天  
```
SELECT * FROM 表名 WHERETO_DAYS(NOW( ) ) - TO_DAYS( 时间字段名) <= 1 
```

##### 1.4 7天  
```
SELECT * FROM 表名 whereDATE_SUB(CURDATE(), INTERVAL 7 DAY) <=date(时间字段名)  
```
##### 1.5 近30天
```  
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <=date(时间字段名)  
```
##### 1.6 本月  
```
SELECT * FROM 表名 WHEREDATE_FORMAT( 时间字段名, '%Y%m' ) =DATE_FORMAT( CURDATE( ) , '%Y%m' )  
```
##### 1.7 上一月  
```
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1  
```

##### 1.8 查询本季度数据  
```
select * from `ht_invoice_information` whereQUARTER(create_date)=QUARTER(now()); 
``` 
##### 1.9 查询上季度数据 
``` 
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));  
```
##### 1.10 查询本年数据  
```
select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW());  
```
> > #查询上年数据  
select * from `ht_invoice_information` where year(create_date)=year(date_sub(now(),interval 1 year));  
> 查询当前这周的数据   
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now());  
> 查询上周的数据  
SELECT name,submittime FROM enterprise WHEREYEARWEEK(date_format(submittime,'%Y-%m-%d')) =YEARWEEK(now())-1;  
查询当前月份的数据  
select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(now(),'%Y-%m')  
查询距离当前现在6个月的数据  
select name,submittime from enterprise where submittime between date_sub(now(),interval 6 month) and now();  
查询上个月的数据  
select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(DATE_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m')  
select * from ` user ` where DATE_FORMAT(pudate, ' %Y%m ' ) = DATE_FORMAT(CURDATE(), ' %Y%m ' ) ;  
select * from user where WEEKOFYEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = WEEKOFYEAR(now())  
select *   
from user
where MONTH (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = MONTH (now())  
select *   
from [ user ]   
where YEAR (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = YEAR (now())  
and MONTH (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = MONTH (now())  
select *   
from [ user ]   
where pudate between 上月最后一天  
and 下月第一天  
where date(regdate)   =   curdate();  
select   *   from   test   where year(regdate)=year(now())   and month(regdate)=month(now())   and day(regdate)=day(now())  
SELECT date( c_instime ) ,curdate( )  
FROM `t_score`  
WHERE 1  
LIMIT 0 , 30 
```
## 二、相关函数简介

 

### 0、Sql server中DateDiff()用法
 

DATEDIFF 函数 [日期和时间]

功能 
返回两个日期之间的间隔。

语法 
DATEDIFF ( date-part, date-expression-1, date-expression-2 )

date-part :
year | quarter | month | week | day | hour | minute | second | millisecond

参数 
date-part    指定要测量其间隔的日期部分。

有关日期部分的详细信息，请参见日期部分。

date-expression-1    某一间隔的起始日期。从 date-expression-2 中减去该值，返回两个参数之间 date-parts 的天数。

date-expression-2    某一间隔的结束日期。从该值中减去 Date-expression-1，返回两个参数之间 date-parts 的天数。

用法 
此函数计算两个指定日期之间日期部分的数目。结果为日期部分中等于（date2 - date1）的有符号的整数值。

当结果不是日期部分的偶数倍时，DATEDIFF 将被截断而不是被舍入。

当使用 day 作为日期部分时，DATEDIFF 返回两个指定的时间之间（包括第二个日期但不包括第一个日期）的午夜数。

当使用 month 作为日期部分时，DATEDIFF 返回两个日期之间（包括第二个日期但不包括第一个日期）出现的月的第一天的数目。

当使用 week 作为日期部分时，DATEDIFF 返回两个日期（包括第二个日期但不包括第一个日期）之间星期日的数目。

对于更小的时间单位存在溢出值：

milliseconds    24 天

seconds    68 年

minutes    4083 年

others    没有溢出限制

如果超出这些限制，此函数将返回溢出错误。

标准和兼容性 
SQL/92    Transact-SQL 扩展。

SQL/99    Transact-SQL 扩展。

Sybase    与 Adaptive Server Enterprise 兼容。

下面示例的语句返回 1：

SELECT datediff( hour, '4:00AM', '5:50AM' )下面的语句返回 102：

SELECT datediff( month, '1987/05/02', '1995/11/15' )下面的语句返回 0：

SELECT datediff( day, '00:00', '23:59' )下面的语句返回 4：

SELECT datediff( day,
   '1999/07/19 00:00',
   '1999/07/23 23:59' )下面的语句返回 0：

SELECT datediff( month, '1999/07/19', '1999/07/23' )下面的语句返回 1：

SELECT datediff( month, '1999/07/19', '1999/08/23' )

 

 

1、MySQLDATE_SUB()函数
MySQL Date 函数

定义和用法
DATE_SUB() 函数从日期减去指定的时间间隔。

语法
DATE_SUB(date,INTERVAL expr type)
date 参数是合法的日期表达式。expr 参数是您希望添加的时间间隔。

type 参数可以是下列值：

Type 值
MICROSECOND
SECOND
MINUTE
HOUR
DAY
WEEK
MONTH
QUARTER
YEAR
SECOND_MICROSECOND
MINUTE_MICROSECOND
MINUTE_SECOND
HOUR_MICROSECOND
HOUR_SECOND
HOUR_MINUTE
DAY_MICROSECOND
DAY_SECOND
DAY_MINUTE
DAY_HOUR
YEAR_MONTH
实例
假设我们有如下的表：

OrderId	ProductName	OrderDate
1	'Computer'	2008-12-29 16:25:46.635
现在，我们希望从 "OrderDate" 减去 2 天。

我们使用下面的 SELECT 语句：

SELECT OrderId,
DATE_SUB(OrderDate,INTERVAL 2 DAY)
 AS OrderPayDate
FROM Orders
结果：

OrderId	OrderPayDate
1	2008-12-27 16:25:46.635
  例2.SELECT DATE_SUB('2010-08-12', INTERVAL '3-2' YEAR_MONTH) AS NewDate  
   结果：2007-06-12
  例3.SELECT DATE_SUB('2011-09-14 2:44:36', INTERVAL '2:26' HOUR_MINUTE) AS NewDate  

   结果： 2011-09-1400:18:36

--------------------------------------------------------------------------------------------------------------------------------

2、TO_DAYS(date)

给定一个日期date, 返回一个天数 (从年份0000-00-00开始的天数 )。
   例：

select TO_DAYS(NOW());
+----------------+
| TO_DAYS(NOW()) |
+----------------+
| 735159 |
+----------------+

select TO_DAYS(121018);
+-----------------+
| TO_DAYS(121018) |
+-----------------+
| 735159 |
+-----------------+

给定一个日期date，返回一个天数（从年份0开始的天数）。

TO_DAYS() 不用于阳历出现(1582)前的值，原因是当日历改变时，遗失的日期不会被考虑在内。

请记住， mysql“日期和时间类型”中的规则将日期中的二位数年份值转化为四位。例如，  ‘1997-10-07′和 ‘97-10-07′ 被视为同样的日期:

对于1582 年之前的日期(或许在其它地区为下一年 ), 该函数的结果实不可靠的。

 

3、MySQL DATE() 函数
MySQL Date 函数

定义和用法
DATE() 函数返回日期或日期/时间表达式的日期部分。

 

 

实例
 

 

假设我们有如下的表：

 

OrderId	ProductName	OrderDate
1	'Computer'	2008-12-29 16:25:46.635
 

 

我们使用下面的 SELECT 语句：

 

SELECT ProductName, DATE(OrderDate) AS OrderDate
FROM Orders
WHERE OrderId=1
 

 

结果：

 

ProductName	OrderDate
'Computer'	2008-12-29

4、MySQL NOW() 函数
MySQL Date 函数

定义和用法
NOW() 函数返回当前的日期和时间。

语法
NOW()
实例
例子 1
下面是 SELECT 语句：

SELECT NOW(),CURDATE(),CURTIME()
结果类似：

NOW()	CURDATE()	CURTIME()
2008-12-29 16:25:46	2008-12-29	16:25:46
例子 2
下面的 SQL 创建带有日期时间列 (OrderDate) 的 "Orders" 表：

CREATE TABLE Orders 
(
OrderId int NOT NULL,
ProductName varchar(50) NOT NULL,
OrderDate datetime NOT NULL DEFAULT NOW(),
PRIMARY KEY (OrderId)
)
请注意，OrderDate 列规定 NOW() 作为默认值。作为结果，当您向表中插入行时，当前日期和时间自动插入列中。

现在，我们希望在 "Orders" 表中插入一条新记录：

INSERT INTO Orders (ProductName) VALUES ('Computer')
"Orders" 表将类似这样：

OrderId	ProductName	OrderDate
1	'Computer'	2008-12-29 16:25:46.635
5、MySQL DATE_FORMAT() 函数
MySQL Date 函数

定义和用法
DATE_FORMAT() 函数用于以不同的格式显示日期/时间数据。

语法
DATE_FORMAT(date,format)
date 参数是合法的日期。format 规定日期/时间的输出格式。

可以使用的格式有：

格式	描述
%a	缩写星期名
%b	缩写月名
%c	月，数值
%D	带有英文前缀的月中的天
%d	月的天，数值(00-31)
%e	月的天，数值(0-31)
%f	微秒
%H	小时 (00-23)
%h	小时 (01-12)
%I	小时 (01-12)
%i	分钟，数值(00-59)
%j	年的天 (001-366)
%k	小时 (0-23)
%l	小时 (1-12)
%M	月名
%m	月，数值(00-12)
%p	AM 或 PM
%r	时间，12-小时（hh:mm:ss AM 或 PM）
%S	秒(00-59)
%s	秒(00-59)
%T	时间, 24-小时 (hh:mm:ss)
%U	周 (00-53) 星期日是一周的第一天
%u	周 (00-53) 星期一是一周的第一天
%V	周 (01-53) 星期日是一周的第一天，与 %X 使用
%v	周 (01-53) 星期一是一周的第一天，与 %x 使用
%W	星期名
%w	周的天 （0=星期日, 6=星期六）
%X	年，其中的星期日是周的第一天，4 位，与 %V 使用
%x	年，其中的星期一是周的第一天，4 位，与 %v 使用
%Y	年，4 位
%y	年，2 位
实例
下面的脚本使用 DATE_FORMAT() 函数来显示不同的格式。我们使用 NOW() 来获得当前的日期/时间：
```
DATE_FORMAT(NOW(),'%b %d %Y %h:%i %p')
DATE_FORMAT(NOW(),'%m-%d-%Y')
DATE_FORMAT(NOW(),'%d %b %y')
DATE_FORMAT(NOW(),'%d %b %Y %T:%f')

结果类似：

Dec 29 2008 11:45 PM
12-29-2008
29 Dec 08
29 Dec 2008 16:25:46.635
6、MySQL获取季度的函数QUARTER(d)
```
 

QUARTER(d)函数返回日期d是一年中的第几季度。值的范围是1～4。

实例
使用QUARTER()函数返回指定日期对应的季度。SQL语句如下：

mysql>SELECT QUARTER('14-09-29');
执行结果如下：

MySQL中使用QUARTER()函数返回指定日期对应的季度的操作效果

从上图中代码执行的结果可以看出，14年9月29日是2014年的第3个季度。

提示
“00～69”转换为“2000～2069”，“70～99”转换为“1970～1999”。

7、MySQL 的 YEARWEEK()函数：

它是获取年份和周数的一个函数，

函数形式为 

YEARWEEK(date[,mode])

例如 2010-3-14 ，礼拜天

SELECT YEARWEEK('2010-3-14') 

SELECT YEARWEEK('2010-3-14',1) 

```
其中第二个参数是 mode ，具体指的意思如下：

Mode	First day of week	Range	Week 1 is the first week …
0	Sunday	0-53	with a Sunday in this year
1	Monday	0-53	with more than 3 days this year
2	Sunday	1-53	with a Sunday in this year
3	Monday	1-53	with more than 3 days this year
4	Sunday	0-53	with more than 3 days this year
5	Monday	0-53	with a Monday in this year
6	Sunday	1-53	with more than 3 days this year
7	Monday	1-53	with a Monday in this year
```