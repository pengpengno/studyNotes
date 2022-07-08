---
title: linux下查看mysql 日志
date: 2021-08-28 09:41:21.426
updated: 2022-05-12 13:18:53.139
url: /archives/linux下查看mysql日志
categories: 
- linux | 数据库
tags: 
---



mysql有以下几种日志：  
   错误日志：     -log-err  
   查询日志：     -log  
   慢查询日志:   -log-slow-queries  
   更新日志:     -log-update  
   二进制日志： -log-bin  


是否启用了日志 
```
mysql>show variables like 'log_%'; 
```
![image-1652332606599](/upload/2022/05/image-1652332606599.png)
怎样知道当前的日志 
```
mysql> show master status; 
```
![image-1652332620298](/upload/2022/05/image-1652332620298.png)
显示二進制日志數目 
```
mysql> show master logs; 
```
![image-1652332630589](/upload/2022/05/image-1652332630589.png)
在mysql中查看二进制文件
```
查询最近所有的
 show binlog events;
 查看指定的文件
 show binlog events in 'mysql-bin.000925'
```

看二进制日志文件用mysqlbinlog 
```shell>mysqlbinlog mail-bin.000001 
或者shell>mysqlbinlog mail-bin.000001 | tail 
```

在配置文件中指定log的輸出位置. 
Windows：Windows 的配置文件为 my.ini，一般在 MySQL 的安装目录下或者 c:\Windows 下。 
Linux：Linux 的配置文件为 my.cnf ，一般在 /etc 下。 

> linux下： 
> 
Sql代码   
 在[mysqld] 中輸入  
#log  
log-error=/usr/local/mysql/log/error.log  
log=/usr/local/mysql/log/mysql.log  
long_query_time=2  
log-slow-queries= /usr/local/mysql/log/slowquery.log  

> windows下: 

 在[mysqld] 中輸入  
```
#log  
log-error="E:/PROGRA~1/EASYPH~1.0B1/mysql/logs/error.log"  
log="E:/PROGRA~1/EASYPH~1.0B1/mysql/logs/mysql.log"  
long_query_time=2  
log-slow-queries= "E:/PROGRA~1/EASYPH~1.0B1/mysql/logs/slowquery.log"  
```

开启慢查询 
```
long_query_time =2  --是指执行超过多久的sql会被log下来，这里是2秒 
log-slow-queries= /usr/local/mysql/log/slowquery.log  --将查询返回较慢的语句进行记录 

log-queries-not-using-indexes = nouseindex.log  --就是字面意思，log下来没有使用索引的query 

log=mylog.log  --对所有执行语句进行记录
```


查看服务启动失败日志 
```
journalctl -xe
```