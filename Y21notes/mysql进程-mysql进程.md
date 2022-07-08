---
title: mysql进程
date: 2022-05-12 13:00:33.991
updated: 2022-05-12 13:05:02.112
url: /archives/mysql进程
categories: 
tags: 
---

查看mysql进程有两种方法

1.进入mysql/bin目录下输入mysqladmin processlist;

2.启动mysql，输入show processlist;

如果有SUPER权限，则可以看到全部的线程，否则，只能看到自己发起的线程(这是指，当前对应的MySQL帐户运行的线程)。
```
mysql> show processlist;
```

![image-1652331702766](/upload/2022/05/image-1652331702766.png)
```


第一列 id，进程一个标识，你要kill一个语句的时候很有用。
eg.    kill 11234;

第二列 user列，显示单前用户，如果不是root，这个命令就只显示你权限范围内的sql语句。

第三列 host列，显示这个语句是从哪个ip的哪个端口上发出的。可以用来追踪出问题语句的用户。

第四列 db列，显示这个进程目前连接的是哪个数据库。

第五列 command列，显示当前连接的执行的命令，一般就是休眠(sleep)，查询(query)，连接(connect)。

第六列 time列，此这个状态持续的时间，单位是秒。

第七列 state列，显示使用当前连接的sql语句的状态，很重要的列，后续会有所有的状态的描述，请注意，state只是语句执行中的某一个状态，一个sql语句，已查询为例，可能需要经过copying to tmp table，Sorting result，Sending data等状态才可以完成。

第八列 info列，显示这个sql语句，因为长度有限，所以长的sql语句就显示不全，但是一个判断问题语句的重要依据。
```
mysql手册里有所有状态的说明，链接如下：http://dev.mysql.com/doc/refman/5.0/en/general-thread-states.html

杀死进程kill id
