---
title: git 使用手册
date: 2017-04-27 09:33:15
tags: [git,使用手册]
categories: git
---

> git 使用手册

<!-- More-->

## 0、 示意图
![image](http://image.beekka.com/blog/2014/bg2014061202.jpg)

## 一、常用命令
### 1、 删除版本库中的一个commit: `git reset --hard HEAD~1`
### 2、 删除远程库中的一个commit: `git reset --hard HEAD~1`, `git push --force`
### 3、 将本地分支推送到远程分支：`git push origin local_ branch:remote_ branch` ：这个操作，local_ branch必须为你本地存在的分支，remote_ branch为远程分支，如果remote_ branch不存在则会自动创建分支。
### 4、将本地分支与远程分支建立对应关系：`git branch --set-upstream-to=origin/<branch> <remote>`  
### 5、将本地分支删除：`git branch -d <branchName>`
### 6、如果对应outlook密码修改，则可以在目录`/Users/username/.ssh`下删除`id_rsa`和`id_rsa.pub`
### 7、切换分支:`git checkout <branchname>`
### 8、切换分支并创建本地分支:`git checkout -b <branchname>`