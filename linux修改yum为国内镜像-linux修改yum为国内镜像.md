---
title: linux修改yum为国内镜像
date: 2021-12-21 05:56:04.975
updated: 2022-04-27 16:35:45.598
url: /archives/linux修改yum为国内镜像
categories: 
tags: 
---



## linux修改yum为国内镜像
#### centos7 修改yum源为阿里源
首先是到yum源设置文件夹里
1. 查看yum源信息:
    yum repolist
2. 定位到base reop源位置
     cd /etc/yum.repos.d
3. 接着备份旧的配置文件
   sudo mv CentOS-Base.repo CentOS-Base.repo.bak
4. 下载阿里源的文件
 sudo wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
---------------------------红帽版本一样的安装------------------------------
```
# 安装epel repo源：
epel(RHEL 7) 红帽7
   wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
epel(RHEL 6) 红帽6
    wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
epel(RHEL 5) 红帽5
    wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-5.repo
```
------------------------------------------------------------------------------    
5.清理缓存
    yum clean all
6.重新生成缓存
    yum makecache
7. 再次查看yum源信息
   yum repolist

如遇上述操作不可用则将etc/yum.respo.d下所有文件备份后清空  并重新执行上述流程