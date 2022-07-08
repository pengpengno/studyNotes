---
title: mysql 日期相关
date: 2021-11-26 16:04:23.652
updated: 2022-04-27 16:35:45.613
url: /archives/mysql日期相关
categories: 
- 数据库
tags: 
- MYSQL
---



[TOC]

##### 查询一天：
```
select * from table where to_days(column_time) =to_days(now());select * from table where date(column_time) = curdate();
```
##### 查询一周：
```
select * from table where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(column_time);
```
##### 查询一个月：
```
select * from table where DATE_SUB(CURDATE(), INTERVAL INTERVAL 1 MONTH) <= date(column_time);
```
##### mysql的日期和时间函数

##### 查询选择所有 date_col 值在最后 30 天内的记录。

mysql> SELECT something FROM tbl_name WHERE TO_DAYS(NOW()) - TO_DAYS(date_col) <= 30; //真方便,以前都是自己写的,竟然不知道有这,失败.

##### DAYOFWEEK(date)

返回 date 的星期索引(1 = Sunday, 2 = Monday, ... 7 = Saturday)。索引值符合 ODBC 的标准。
```
mysql> SELECT DAYOFWEEK(’1998-02-03’);-> 3
```
##### WEEKDAY(date)

返回 date 的星期索引(0 = Monday, 1 = Tuesday, ... 6 = Sunday)：
```
mysql> SELECT WEEKDAY(’1998-02-03 22:23:00’);-> 1mysql> SELECT WEEKDAY(’1997-11-05’);-> 2
```
DAYOFMONTH(date)

返回 date 是一月中的第几天，范围为 1 到 31：

mysql> SELECT DAYOFMONTH(’1998-02-03’);-> 3

##### DAYOFYEAR(date)
```
返回 date 是一年中的第几天，范围为 1 到 366：

mysql> SELECT DAYOFYEAR(’1998-02-03’);-> 34
```
##### MONTH(date)
```
返回 date 中的月份，范围为 1 到 12：

mysql> SELECT MONTH(’1998-02-03’);-> 2
```
##### DAYNAME(date)
```
返回 date 的星期名：

mysql> SELECT DAYNAME("1998-02-05");-> ’Thursday’
```
MONTHNAME(date)

返回 date 的月份名：
```
mysql> SELECT MONTHNAME("1998-02-05");-> ’February’
```
QUARTER(date)
```
返回 date 在一年中的季度，范围为 1 到 4：
```
mysql> SELECT QUARTER(’98-04-01’);-> 2WEEK(date)

WEEK(date,first)

对于星期日是一周中的第一天的场合，如果函数只有一个参数调用，返回 date 为一年的第几周，返回值范围为 0 到 53 (是的，可能有第 53 周的开始)。两个参数形式的 WEEK() 允许你指定一周是否以星期日或星期一开始，以及返回值为 0-53 还是 1-52。 这里的一个表显示第二个参数是如何工作的：

值 含义
```
0 一周以星期日开始，返回值范围为 0-53

1 一周以星期一开始，返回值范围为 0-53

2 一周以星期日开始，返回值范围为 1-53

3 一周以星期一开始，返回值范围为 1-53 (ISO 8601)

mysql> SELECT WEEK(’1998-02-20’);-> 7mysql> SELECT WEEK(’1998-02-20’,0);-> 7mysql> SELECT WEEK(’1998-02-20’,1);-> 8mysql> SELECT WEEK(’1998-12-31’,1);-> 53
```
注意，在版本 4.0 中，WEEK(#,0) 被更改为匹配 USA 历法。 注意，如果一周是上一年的最后一周，当你没有使用 2 或 3 做为可选参数时，MySQL 将返回 0：
```
mysql> SELECT YEAR(’2000-01-01’), WEEK(’2000-01-01’,0);-> 2000, 0mysql> SELECT WEEK(’2000-01-01’,2);-> 52
```
你可能会争辩说，当给定的日期值实际上是 1999 年的第 52 周的一部分时，MySQL 对 WEEK() 函数应该返回 52。我们决定返回 0 ，是因为我们希望该函数返回“在指定年份中是第几周”。当与其它的提取日期值中的月日值的函数结合使用时，这使得 WEEK() 函数的用法可靠。 如果你更希望能得到恰当的年-周值，那么你应该使用参数 2 或 3 做为可选参数，或者使用函数 YEARWEEK() ：
```
mysql> SELECT YEARWEEK(’2000-01-01’);-> 199952mysql> SELECT MID(YEARWEEK(’2000-01-01’),5,2);-> 52
```
##### YEAR(date)
```
返回 date 的年份，范围为 1000 到 9999：

mysql> SELECT YEAR(’98-02-03’);-> 1998
```
YEARWEEK(date)

##### YEARWEEK(date,first)
```
返回一个日期值是的哪一年的哪一周。第二个参数的形式与作用完全与 WEEK() 的第二个参数一致。注意，对于给定的日期参数是一年的第一周或最后一周的，返回的年份值可能与日期参数给出的年份不一致：

mysql> SELECT YEARWEEK(’1987-01-01’);-> 198653

注意，对于可选参数 0 或 1，周值的返回值不同于 WEEK() 函数所返回值(0)， WEEK() 根据给定的年语境返回周值。
```
##### HOUR(time)
```
返回 time 的小时值，范围为 0 到 23：

mysql> SELECT HOUR(’10:05:03’);-> 10
```
##### MINUTE(time)
```
返回 time 的分钟值，范围为 0 到 59：

mysql> SELECT MINUTE(’98-02-03 10:05:03’);-> 5
```
##### SECOND(time)
```
返回 time 的秒值，范围为 0 到 59：

mysql> SELECT SECOND(’10:05:03’);-> 3
```
##### PERIOD_ADD(P,N)
```
增加 N 个月到时期 P(格式为 YYMM 或 YYYYMM)中。以 YYYYMM 格式返回值。 注意，期间参数 P 不是 一个日期值：

mysql> SELECT PERIOD_ADD(9801,2);-> 199803

##### PERIOD_DIFF(P1,P2)

返回时期 P1 和 P2 之间的月数。P1 和 P2 应该以 YYMM 或 YYYYMM 指定。 注意，时期参数 P1 和 P2 不是 日期值：

mysql> SELECT PERIOD_DIFF(9802,199703);-> 11
```
DATE_ADD(date,INTERVAL expr type)

DATE_SUB(date,INTERVAL expr type)

ADDDATE(date,INTERVAL expr type)

SUBDATE(date,INTERVAL expr type)