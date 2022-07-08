---
title: Docker部署
date: 2021-12-18 15:20:02.066
updated: 2022-04-27 16:35:38.575
url: /archives/docker部署
categories: 
- Java | Docker
tags: 
- linux | Docker | Spring | 开发开发开发！！！
---



# Docker
[TOC]
## 100 Linux 环境安装Docker
###  110、Docker安装与配置
> 首先确认当前服务器的yum源搭建完毕[Linux 修改yum源](http://180.76.240.8:8090/archives/linux-xiu-gai-yum-wei-guo-nei-jing-xiang)
#### 1.安装依赖包
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 
```
#### 2.设置阿里云镜像源
```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 


或者使用下方链接
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

```
#### 3.安装 Docker-CE
```
重建 Yum 缓存。

安装 Docker-CE ，请执行一下命令进行安装：

sudo yum install docker-ce
```
#### 4.启动 Docker-CE
```
sudo systemctl enable docker
sudo systemctl start docker
```
#### 5.[可选] 为 Docker 建立用户组
```
docker 命令与 Docker 引擎通讯之间通过 UnixSocket ，但是能够有权限访问 UnixSocket 的用户只有 root 和 docker 用户组的用户才能够进行访问，所以我们需要建立一个 docker 用户组，并且将需要访问 docker 的用户添加到这一个用户组当中来。
```
##### 5.1 建立 Docker 用户组
``` 
sudo groupadd docker
```
##### 5.2.添加当前用户到 docker 组
```
sudo usermod -aG docker $USER
```
#### 6.镜像加速配置
> 这里使用的是 阿里云提供的镜像加速 ，登录并且设置密码之后在左侧的 Docker Hub 镜像站点 可以找到专属加速器地址，复制下来。

然后执行以下命令：
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json 
{
  "registry-mirrors": ["你的加速器地址"]
}```
sudo systemctl daemon-reload
sudo systemctl restart docker
> 之后重新加载配置，并且重启 Docker 服务

systemctl daemon-reload
systemctl restart docker


sudo yum install -y yum-utils device-mapper-persistent-data lvm2 
```

```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
```
### 120 配置Docker 的远程
配置远程访问
#### 编辑docker服务配置文件
```
sudo vim /lib/systemd/system/docker.service
找到如下配置

ExecStart=/usr/bin/dockerd
修改为


ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```
## 200 Docker的基本指令
### 210 Docker 常用命令
#### 1.拉取镜像
```
docker pull
```
#### 2.删除容器
```
docker rm <容器名 or ID>
```
#### 3.查看容器日志
```
docker logs -f <容器名 or ID>
```
#### 4.查看正在运行的容器
```
docker ps
docker ps -a # 为查看所有的容器，包括已经停止的。
```
#### 5.删除所有容器
```
docker rm $(docker ps -a -q)
```
#### 6.停止、启动、杀死指定容器
```
docker start <容器名 or ID> # 启动容器
docker stop <容器名 or ID> # 启动容器
docker kill <容器名 or ID> # 杀死容器
```
#### 7.查看所有镜像
```
docker images
```
#### 8.拉取镜像
```
docker pull <镜像名:tag>
# 例如以下代码
docker pull sameersbn/redmine:latest
```
#### 9.后台运行
```
docker run -d <Other Parameters>
# 例如
docker run -d -p 127.0.0.1:33301:22 centos6-ssh
```
#### 10.暴露端口
```
# 一共有三种形式进行端口映射
docker -p ip:hostPort:containerPort # 映射指定地址的主机端口到容器端口
# 例如：docker -p 127.0.0.1:3306:3306 映射本机3306端口到容器的3306端口
docker -p ip::containerPort # 映射指定地址的任意可用端口到容器端口
# 例如：docker -p 127.0.0.1::3306 映射本机的随机可用端口到容器3306端口
docer -p hostPort:containerPort # 映射本机的指定端口到容器的指定端口
# 例如：docker -p 3306:3306 # 映射本机的3306端口到容器的3306端口
```
#### 11.映射数据卷
```
docker -v /home/data:/opt/data # 这里/home/data 指的是宿主机的目录地址，后者则是容器的目录地址
```
210 ### 使用DockerHub 进行镜像的上传下载

#### 1.容器的打包
```
docker commit  -a  "here is pengpeng"   -m "first commit" 容器Id 要打包成的镜像名称
```
#### 2. 将 镜像tag
```
docker  tag  镜像Id  [你的dockerhubId]/[仓库名称]

docker tag 3123h1ghf :v1 pengepeng163/myblog
```

#### 3. 上传镜像
```
docker push [镜像名称]:[版本]

docker push pengpeng/myblog:v1
```
#### 4.  下载镜像文件
```
docker pull pengpeng163/myblog
```
> ps 如果需要从其他网站下载镜像
```
例如，docker pull ubuntu:18.04命令相当于docker pull registry.hub.docker.com/ubuntu:18.04

如果从非官方的仓库下载，则要在仓库名称前指定完整的仓库地址。例如从网易蜂巢的镜像源来下载ubuntu:18.04镜像，可以使用如下命令，此时下载的镜像名称为

hub.c.163.com/public/ubuntu:18.04：

拉取语句为

$ docker pull hub.c.163.com/public/ubuntu:18.04
```
#### GUI 管理配置
```
这里推荐使用 Portainer 作为容器的 GUI 管理方案。

官方地址：https://portainer.io/install.html

安装命令：

docker volume create portainer_data
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
访问你的 IP:9000 即可进入容器管理页面。
```

## 300 IDEA中使用Docker部署工程
Docker 配置图解
![1](http://180.76.240.8:8090/upload/2021/12/1-bfd427be0db545c29ea64d2e3e6cb311.png)

![docker2](http://180.76.240.8:8090/upload/2021/12/docker2-47d20fb79c60416ca23dfab349c35c98.png)
#### 设定启动容器前命令 添加before lanuch
```
clean package -Dmaven.test.skip=true
```
#### 设定Maven pom配置
```

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
<!--                    指定打包钉路径-->
                    <outputDirectory>
                        ${project.basedir}/docker
                    </outputDirectory>
<!--                    <configuration>-->
                        <!-- 指定该Main Class为全局的唯一入口 -->
                        <mainClass>com.peng.docker.DockerApplication</mainClass>
                        <layout>ZIP</layout>
<!--                    </configuration>-->
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

```

#### 编写DocerFile
```
FROM java:8u111

VOLUME /tmp

ADD *.jar app.jar

EXPOSE 80

ENTRYPOINT ["java","-jar","/app.jar"]

# Ubuntu 时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
配置完毕后 直接运行Docker 容器即可

登录服务器进行验证

## 400 QA
### Error: Unable to access jarfile docker.jar
1.DockerFile 中ADD 的jar 路径是否 准确 
2. 当前用户权限是否 为root权限
3. 是否为相对路径
### Could not transfer artifact org.springframework.boot:spring-boot-maven-plugi
1. 排查Maven仓库是否使用镜像地址
2. 安装相应证书
3. 使maven命令 mvn verify install :install
### xxx-1.0-SNAPSHOT.jar中没有主清单属性
1.找不到运行额达主类
pom文件设置
```
打包方式设置为jar
    <packaging>jar</packaging>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
<!--                    <configuration>-->
                        <!-- 指定该Main Class为全局的唯一入口 -->
                        <mainClass>com.peng.docker.DockerApplication</mainClass>
                        <layout>ZIP</layout>
<!--                    </configuration>-->
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### maven 依赖爆红

报红是因为缺少版本号，后面加上