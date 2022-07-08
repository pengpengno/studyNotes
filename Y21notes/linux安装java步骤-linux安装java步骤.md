---
title: linux安装java步骤
date: 2021-10-22 11:26:07.194
updated: 2022-04-27 16:35:33.764
url: /archives/linux安装java步骤
categories: 
tags: 
---



linux安装java步骤
本文转发自博客园-Q鱼丸粗面Q、博客园-郁冬的文章，内容略有改动

本文已收录至博客专栏linux安装各种软件及配置环境教程中

方式一：yum方式下载安装
1、查找java相关的列表

yum -y list java*



或者

yum search jdk



2、安装jdk

yum install java-1.8.0-openjdk.x86_64

3、完成安装后验证

java -version



4、通过yum安装的默认路径为：/usr/lib/jvm



5、将jdk的安装路径加入到JAVA_HOME

vi /etc/profile

在文件最后加入：

#set java environment
JAVA_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME CLASSPATH PATH
修改/etc/profile之后让其生效

. /etc/profile （注意 . 之后应有一个空格）

方式二、官网下载jdk，ftp上传服务器解压安装
1、进入 Oracle 官方网站 下载合适的 JDK 版本，准备安装。
注意：这里需要下载 Linux 版本。这里以jdk-8u151-linux-x64.tar.gz为例，你下载的文件可能不是这个版本，这没关系，只要后缀(.tar.gz)一致即可。

2、创建目录

在/usr/目录下创建java目录，

mkdir /usr/local/java
cd /usr/local/java
把下载的文件 jdk-8u151-linux-x64.tar.gz 放在/usr/local/java/目录下。

3. 解压 JDK

tar -zxvf jdk-8u151-linux-x64.tar.gz

4. 设置环境变量

修改 vi /etc/profile

在 profile 文件中添加如下内容并保存：

set java environment
JAVA_HOME=/usr/local/java/jdk1.8.0_151        
JRE_HOME=/usr/local/java/jdk1.8.0_151/jre     
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
 

注意：其中 JAVA_HOME， JRE_HOME 请根据自己的实际安装路径及 JDK 版本配置。

让修改生效：

source /etc/profile

5. 测试
java -version

显示 java 版本信息，则说明 JDK 安装成功