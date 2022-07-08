---
title: Sdkman的简介
date: 2022-02-19 14:36:37.237
updated: 2022-04-27 16:35:21.657
url: /archives/sdkman的简介
categories: 
tags: 
---



sdkman的介绍、安装及使用
 Linux  fireling  5年前 (2017-02-23)  22492℃
最近接触了一款新的sdk部署工具，sdkman，它的官网地址为 http://sdkman.io/ ，分享给大家~

sdkman介绍：
sdkman(The Software Development Kit Manager)是类unix上的开发工具sdk管理工具，可以方便的管理开发工具sdk(主要是jvm上的)的安装、卸载、版本切换等。

sdkman提供命令行客户端工具，用户可以在客户端通过使用sdk一系列命令，在服务端方便部署sdk环境。它目前支持安装组件如下：

1

sdkman安装：

curl -s "https://get.sdkman.io" | bash
默认的安装目录为~/.sdkman。

2

如果要切换安装目录，可以之前把SDKMAN_DIR预设一下：


export SDKMAN_DIR="/usr/local/sdkman"
激活sdkman：


source ~/.sdkman/bin/sdkman-init.sh
sdkman使用：
通过输入sdk help命令，查看sdk相关命令：

3

命令说明：
List
sdk list
使用这个命令，可以查看sdkman支持安装的sdk组件。
sdk list candidate
查看某个candidate版本，也就是sdkman提供安装的版本。
比如说，sdk list java，可以查看java的可用版本。4
Install
sdk install candidate
安装某个candidate，默认安装最新的stable版本。当然，你可以指定版本号，比如说我要安装7u80版本的java环境，则输入
sdk install java 7u80
Uninstall
sdk uninstalll candidate
卸载某个candidate，同安装一样，也可以指定某个版本卸载。
Use
sdk use candidate
在当前shell环境使用candidate，或者指定某个版本的candidate。退出该shell则环境失效。如果想一直默认改candidate，则使用default命令。
Default
sdk default candidate
默认使用的candidate。
Current
列出所有的candidates或者某个candidate，当前使用的版本。
Outdated
列出过期candidates或某个candidate。
Upgrade
升级所有candidates或某个candidate。
Version
查看版本。
Broadcast
查看最新的sdk发布消息。
Offline
这里可以通过 sdk offline enable 选择 脱机模式，反之选择 联机模式。
Selfupdate
sdkman自我更新。
Flush
刷新一些信息。
sdkman使用举例：
比如说，我要安装java环境和scala环境，则只需要按照如下执行：


sdk install java
5

可以看出，默认的java环境位于~/.sdkman目录下。

6


sdk install scala 
7

sdkman工作原理：
sdkman采用一系列sh脚本，来注册sdk相关命令。

目前支持的组件，可以通过 https://api.sdkman.io/2/candidates/all 查看。

如果系统中通过sdkman安装某candidate，它的安装主要依赖sdkman-install.sh来实现的，依次下载了candidate的zip压缩包及安装脚本，再通过脚本来执行安装流程，本质上是与正常的unix安装流程是一样的。

通过提供的两个接口来进行组件与安装脚本的下载与安装：


https://api.sdkman.io/1/
https://api.sdkman.io/2/
通过制定相关的candidate及version信息，来拼接下载的url，从sdkman官方提供的源头下载。比如说，linux下2.12.1版本的scala下载链接为：https://api.sdkman.io/2/broker/download/scala/2.12.1/linux

下载的组件分别至于archive及temp目录下，安装的组件在candidates目录下，可以看到默认的current为指向某一版本的软链接。

8

每次启动shell环境，都会预先加载 .bash_profile .profile .bashrc .zshrc 等文件中的环境，这样就可以直接使用~/.sdkman中的sdk环境了。