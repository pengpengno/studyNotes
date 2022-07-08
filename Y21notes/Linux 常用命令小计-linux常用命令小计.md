---
title: Linux 常用命令小计
date: 2021-12-03 14:52:31.698
updated: 2022-04-27 16:35:33.739
url: /archives/linux常用命令小计
categories: 
tags: 
---



## Linux 常用命令小计 

（长期更新）
[TOC]

```
>  查找进程 ps -ef | grep xxxxx
```

```
解压命令  
tar -zxvf xxx.tar.gz
```


#### 100 Linux下的解压命令小结
> Linux下常见的压缩包格式有5种:zip tar.gz tar.bz2 tar.xz tar.Z

> 其中tar是种打包格式,gz和bz2等后缀才是指代压缩方式:gzip和bzip2

 

##### filename.zip的解压:
```
unzip filename.zip
 ```

##### filename.tar.gz的解压:
```

tar -zxvf filename.tar.gz
其中zxvf含义分别如下

z: 　　gzip  　　　　　　　　    压缩格式

x: 　　extract　　　　　　　　  解压

v:　　 verbose　　　　　　　　详细信息

f: 　　file(file=archieve)　　　　文件

 ```
##### filename.tar.bz2的解压:
```
tar -jxvf filename.tar.bz2

j: 　　bzip2　　　　　　　　　 压缩格式

其它选项和tar.gz解压含义相同
```
 

##### filename.tar.xz的解压: 
```
tar -Jxvf filename.tar.xz
注意J大写
```
 
##### filename.tar.Z的解压: 
```
tar -Zxvf filename.tar.Z
注意Z大写
```
 
```
关于tar的详细命令可以

tar --help
```
#### 200 linux 端口 进程信息查看
##### 1、根据进程名查看进程信息，以查看tomcat进程名为例，查看所对应的进程id为1095(或者使用： ps -aux | grep tomcat 查看占用内存等信息)
```
ps -ef | grep tomcat
```

##### 2、根据进程id查看进程占用端口，查看对应端口为8080（如果没有netstat命令，使用 yum  -y  install  net-tools安装）
```
netstat -nap | grep 1095
```

##### 3、根据端口查看对应进程，查看占用8080端口的进程id，为1095
```
netstat -tunlp | grep 8080
```

##### 4、根据进程id查看进程信息，查看进程id为1095的进程信息
```
ps -ef | grep 1095
```

##### 5、根据进程id杀死进程，杀死进程id为1095的进程
```
kill -9 1095
```
##### 6、 查看端口占用情况
```
lsof -i:8080
```
```
# window 查看占用端口80
netstat -aon|findstr "80"

# 查询进程号2448信息
tasklist|findstr "2448"

# 可以在任务管理器查询进程号并杀死
```
#### linux用户权限命令
##### 添加用户 
```
adduser  xxxx
```
##### 修改用户密码
```
passwd xxxxxx
```

##### 赋予root权限操作
```
方法一：修改 /etc/sudoers 文件，找到下面一行，把前面的注释（#）去掉

## Allows people in group wheel to run all commands
%wheel    ALL=(ALL)    ALL

然后修改用户，使其属于root组（wheel），命令如下：

#usermod -g root tommy

修改完毕，现在可以用tommy帐号登录，然后用命令 su – ，即可获得root权限进行操作。

方法二：修改 /etc/sudoers 文件，找到下面一行，在root下面添加一行，如下所示：

## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
tommy   ALL=(ALL)     ALL

修改完毕，现在可以用tommy帐号登录，然后用命令 sudo – ，即可获得root权限进行操作。

方法三：修改 /etc/passwd 文件，找到如下行，把用户ID修改为 0 ，如下所示：
tommy:x:0:33:tommy:/data/webroot:/bin/bash
```
####  查看当前登录用户的组内成员
```
groups
```
#### 查看gliethttp用户所在的组,以及组内成员
```
groups gliethttp 
```
#### 查看当前登录用户名
```
whoami
```
## ssh命令
### 生成sshkey
```
ssh-keygen -b 4096 -C "pengpeng_on.com"
```
##### 查看当前生成的公钥（默认路径）
```
cat ~/.ssh/id_rsa.pub

```
##### 在其他主机上添加ssh访问
```
将公钥安装再服务器上
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.100.10

如果目标主机下 ～/.ssh路径下 无 authorized_keys文件
touch authorized_keys

然后再次使用上述命令
```

### 400 文件权限

#### 查看当前文件夹下权限
```
ls  -l
```
> 关于文件权限的解读
![linux文件权限](http://180.76.240.8:8090/upload/2021/12/linux%E6%96%87%E4%BB%B6%E6%9D%83%E9%99%90-3d061caad03f45db883dfae1bac02433.png)
赋予读写用户权限
```
chmod a+wr xxx(文件夹)
```
### 500 网络相关
#### 查看本机ip
```
curl ipconfig.me
```

#### 查看计算机网络相关信息
```
ipconfig
```