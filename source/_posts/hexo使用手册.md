---
title: hexo使用手册
date: 2017-03-19 11:04:18
categories: Hexo
tags: 使用手册
---

> Hexo 的日常使用，包括增加，修改以及多设备操作

<!--More-->

## 一、关于日常笔记（添加新，修改等）：
+ 依次执行git pull, git add .、git commit -m "..."、git push origin hexo指令将改动推送到GitHub；
+ 然后才执行`hexo clean g d`发布网站到master分支上 
+ master分支不用理会，hexo管理   

## 二、其它设备上使用下列步骤：
+ 使用git clone -b hexo https://github.com/xxxx/xxxx.git ；
+ 在本地库中执行：npm install hexo、npm install、npm install hexo-deployer-git（记得，不需要hexo init）

## 三、Git https 转 ssh
### git clone 时一般有两种方式https和ssh，但是https模式下每一次push都需要输入用户名与密码，而ssh方式下只要设置好ssh后不需要每一次都输入用户名与密码。那么如果一个https如何才能转成ssh呢，那就按以下步骤：    
+ 通过`git remote -v`查看当前模式
+ 通过`git remote set-url git@github.com:xxxx/xxx.git`便可转成ssh


## 四、关于hexo使用markdown语法
+ 代码块使用tap键缩进，不要使用语法
+ 标题与`##`之间要空格

## 五、hexo相关图片如果需要站点内部引用则需要放置:`themes/next/source/uploads(images)`