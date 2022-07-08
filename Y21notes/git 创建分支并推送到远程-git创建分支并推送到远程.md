---
title: git 创建分支并推送到远程
date: 2021-11-19 09:56:59.305
updated: 2022-04-27 16:35:38.558
url: /archives/git创建分支并推送到远程
categories: 
- GIt
tags: 
---



## git 创建分支并推送到远程



//创建新的分支：yapple
$git checkout -b yapple

//将新的分支push到远程
$git push origin yapple

//将本地分支关联远程分支
$git branch --set-upstream-to=origin/yapple yapple

//拉取验证
$git pull

或者：
git checkout -b yapple origin/yapple