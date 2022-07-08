---
title: linux 安装docker
date: 2021-10-22 12:09:18.387
updated: 2022-04-27 16:35:43.203
url: /archives/linux安装docker
categories: 
- linux | Docker
tags: 
---



linux 安装docker
1、安装环境
此处在Centos7进行安装，可以使用以下命令查看CentOS版本

lsb_release -a


在 CentOS 7安装docker要求系统为64位、系统内核版本为 3.10 以上，可以使用以下命令查看

uname -r


2、用yum源安装
2.1 查看是否已安装docker列表

yum list installed | grep docker


2.2 安装docker

yum -y install docker
-y表示不询问安装，直到安装成功，安装完后再次查看安装列表



2.3 启动docker

systemctl start docker
2.4 查看docker服务状态

systemctl status docker


以上说明docker安装成功

3、离线安装模式
3.1 安装包官方地址：https://download.docker.com/linux/static/stable/x86_64/

可以先下载到本地，然后通过ftp工具上传到服务器上，或者在服务器上使用命令下载

wget https://download.docker.com/linux/static/stable/x86_64/docker-18.06.3-ce.tgz
3.2 解压

tar -zxvf docker-18.06.3-ce.tgz
3.3 将解压出来的docker文件复制到 /usr/bin/ 目录下

cp docker/* /usr/bin/
3.4 在/etc/systemd/system/目录下新增docker.service文件，内容如下，这样可以将docker注册为service服务

复制代码
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
  
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=127.0.0.1
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
  
[Install]
WantedBy=multi-user.target
复制代码
此处的--insecure-registry=127.0.0.1（此处改成你私服ip）设置是针对有搭建了自己私服Harbor时允许docker进行不安全的访问，否则访问将会被拒绝。

3.5 启动docker

给docker.service文件添加执行权限

chmod +x /etc/systemd/system/docker.service 
重新加载配置文件（每次有修改docker.service文件时都要重新加载下）

systemctl daemon-reload                
启动

systemctl start docker
设置开机启动

systemctl enable docker.service
查看docker服务状态

systemctl status docker


上图表示docker已安装成功