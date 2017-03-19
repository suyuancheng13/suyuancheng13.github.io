---
title: hexo使用手册
date: 2017-03-19 11:04:18
tags:
---

## 二、关于日常笔记（添加新，修改等）
+ 1. 依次执行git pull, git add .、git commit -m "..."、git push origin hexo指令将改动推送到GitHub；
+ 2. 然后才执行hexo g -d发布网站到master分支上 
+ 3. master分支不用理会，hexo管理
## 三、其它设备上使用下列步骤：
+ 1. 使用git clone -b hexo https://github.com/xxxx/xxxx.git ；
+ 2. 在本地库中执行：npm install hexo、npm install、npm install hexo-deployer-git（记得，不需要hexo init）