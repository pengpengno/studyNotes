---
title: Docker 搭建Nexus
date: 2022-03-09 16:52:53.192
updated: 2022-04-27 16:38:42.131
url: /archives/docker搭建nexus
categories: 
tags: 
---





### ###如果没有搭建私服会有什么问题？
如果没有私服，我们所需的所有构件都需要通过 Maven 的中央仓库或者第三方的 Maven 仓库下载到本地，
而一个团队中的所有人都重复的从 Maven 仓库下载构件无疑加大了仓库的负载和浪费了外网带宽，
如果网速慢的话，还会影响项目的进程。
另外，很多情况下项目的开发都是在内网进行的，可能根本连接不了 Maven 的中央仓库和
第三方的 Maven 仓库。
我们开发的公共构件如果需要提供给其它项目使用，也需要搭建私服。

搭建私服的优点
    Maven 私服的概念就是在本地架设一个 Maven 仓库服务器，在代理远程仓库的同时维护本地仓库。
    当我们需要下载一些构件（artifact）时，如果本地仓库没有，再去私服下载，私服没有，
    再去中央仓库下载。这样做会有如下一些优点：
减少网络带宽流量
加速 Maven 构建
部署第三方构件
提高稳定性、增强控制
降低中央仓库的负载

### Nexus 介绍
    Nexus 是一个专门的 Maven 仓库管理软件，它不仅能搭建 Maven 私服，
    还具备如下一些优点使其日趋成为最流行的 Maven 仓库管理器：
提供了强大的仓库管理功能，构件搜索功能
它基于 REST，友好的 UI 是一个 ext.js 的 REST 客户端
它占用较少的内存
基于简单文件系统而非数据库


### 常用的docker命令
```
docker version 查看docker的版本号，包括客户端、服务端、依赖的Go等
docker info 查看系统(docker)层面信息，包括管理的images, containers数等
docker search  在docker index中搜索image
docker pull  从docker registry server 中下拉image
docker push  推送一个image或repository到registry
docker push :TAG 同上，指定tag
docker inspect  查看image或container的底层信息
docker images TODO filter out the intermediate image layers (intermediate image layers 是什么)
docker images -a 列出所有的images
docker ps 默认显示正在运行中的container
docker ps -l 显示最后一次创建的container，包括未运行的
docker ps -a 显示所有的container，包括未运行的
docker logs  查看container的日志，也就是执行命令的一些输出
docker rm  删除一个或多个container
docker rm `docker ps -a -q` 删除所有的container
docker ps -a -q | xargs docker rm 同上, 删除所有的container
docker rmi  删除一个或多个image
docker start/stop/restart  开启/停止/重启container
docker start -i  启动一个container并进入交互模式
docker attach  attach一个运行中的container
docker run  使用image创建container并执行相应命令，然后停止
docker run -i -t  /bin/bash 使用image创建container并进入交互模式, login shell是/bin/bash
docker run -i -t -p  将container的端口映射到宿主机的端口
docker commit  [repo:tag] 将一个container固化为一个新的image，后面的repo:tag可选
docker build
 寻找path路径下名为的Dockerfile的配置文件，使用此配置生成新的image
docker build -t repo[:tag] 同上，可以指定repo和可选的tag
docker build -  使用指定的dockerfile配置文件，docker以stdin方式获取内容，使用此配置生成新的image
docker port  查看本地哪个端口映射到container的指定端口，其实用docker ps 也可以看到

```

通过 exec 命令对指定的容器执行 bash
docker exec -it 550dd77a89e1 /bin/bash
docker start [containerid] 启动docker容器（如果docker的容器手动kill了）
service docker start 启动docker

设置docker开机自启动
chkconfig docker on

使用docker 安装
1，首先执行如下命令下载 Nexus3 镜像：

docker pull sonatype/nexus3
1
2，接着执行如下命令，创建宿主机挂载目录：

mkdir –vp /usr/local/nexus-data
1
3，最后执行如下命令运行 Nexus3 容器即可：

docker run -d --name nexus3 -p 8081:8081 -v /usr/local/nexus-data:/var/nexus-data sonatype/nexus3
1
4，本机防火墙 不要忘记执行如下命令开放 8081 端口：

firewall-cmd --permanent --add-port=8081/tcp
firewall-cmd --reload
1
2
5，通过 exec 命令对指定的容器执行 bash,查看 admin的密码
```
docker exec -it 550dd77a89e1 /bin/bash
vi /nexus-data/admin.password
exit
```
6,访问搭建的nexus后台管理页面 http://ip:8081


7，登录后的界面如下：
（1）默认仓库说明：

maven-central：maven 中央库，默认从 https://repo1.maven.org/maven2/ 拉取 jar
maven-releases：私库发行版 jar，初次安装请将 Deployment policy 设置为 Allow redeploy
maven-snapshots：私库快照（调试版本）jar
maven-public：仓库分组，把上面三个仓库组合在一起对外提供服务，在本地 maven 基础配置 settings.xml 或项目 pom.xml 中使用
1
2
3
4
（2）仓库类型说明：

group：这是一个仓库聚合的概念，用户仓库地址选择 Group 的地址，即可访问 Group 中配置的，用于方便
开发人员自己设定的仓库。maven-public 就是一个 Group 类型的仓库，内部设置了多个仓库，访问顺序取决
于配置顺序，3.x 默认为 Releases、Snapshots、Central，当然你也可以自己设置。

hosted：私有仓库，内部项目的发布仓库，专门用来存储我们自己生成的 jar 文件
snapshots：本地项目的快照仓库
releases： 本地项目发布的正式版本
proxy：代理类型，从远程中央仓库中寻找数据的仓库（可以点击对应的仓库的 Configuration 页签下
 Remote Storage 属性的值即被代理的远程仓库的路径），如可配置阿里云 maven 仓库
central：中央仓库
1
2
3
4
5
6
7
8
9
10
创建一个代理私有仓库（proxy）




aliyun-url : http://maven.aliyun.com/nexus/content/groups/public/



配置 mavne-public 加入列表并配置新加的proxy的顺序


复制你的maven nexus 地址


通过 pom.xml 文件配置新的仓库

<repositories>
    <repository>
        <id>maven-nexus</id>
        <name>maven-nexus</name>
        <url>http://192.168.0.129:8081/repository/maven-public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>

也可以配置阿里云 maven 仓库

<repositories>
   <repository>
      <id>maven-aliyun</id>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <releases>
         <enabled>true</enabled>
      </releases>
      <snapshots>
         <enabled>true</enabled>
         <updatePolicy>always</updatePolicy>
         <checksumPolicy>fail</checksumPolicy>
      </snapshots>
   </repository>
</repositories>

Maven 配置使用私服（下载插件）
下面是一个使用 pom.xml 配置样例：

<pluginRepositories>
    <pluginRepository>
        <id>maven-nexus</id>
        <name>maven-nexus</name>
        <url>http://192.168.0.129:8081/nexus/repository/maven-public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>

Maven 配置使用私服（发布依赖）
（1）首先修改 setting.xml 文件，指定 releases 和 snapshots server 的用户名和密码：

<servers>
    <server>
        <id>releases</id>
        <username>admin</username>
        <password>123</password>
    </server>
    <server>
        <id>snapshots</id>
        <username>admin</username>
        <password>123</password>
    </server>
</servers>

（2）接着在项目的 pom.xml 文件中加入 distributionManagement 节点：
注意：repository 里的 id 需要和上一步里的 server id 名称保持一致。

<distributionManagement>
    <repository>
        <id>releases</id>
        <name>Releases</name>
        <url>http://192.168.0.129:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Snapshot</name>
        <url>http://192.168.0.129:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
