---
title: Github分支备份hexo
date: 2019-07-05 12:00:00
author: dylan
top: false
cover: false
password: 
categories: Blog
tags: 
    - Blog
    - hexo
    - 备份
    - 分支
    - GitHub
---

# 一、前言
使用hexo搭建个人博客框架，配置起来有些消耗时间，管理起来也不是特别方便。特别是有时需要在其他电脑上写博客时，就让人头疼。所以我们就利用Github的分支，来备份hexo，方便快速搭上手写博客。
***

# 二、创建本地分支目录
## 1. 新建文件夹存放分支工作目录。
`mkdir hexo`

## 2. 把你的GitHub的远程仓库克隆到hexo文件夹
`git clone https://github.com/yourusername/yourusername.github.io hexo`

## 3. 删除除了版本管理的.git之外的所有文件和文件夹
```
cd hexo
rm -r *
```

## 4. 把要备份的文件复制到hexo目录
```
scaffolds/
source/
themes/
.git/
.gitignore
_config.yml
package.json
```

>注意：
>如果使用的主题是从Github克隆的，那么使用命令删除它的Git文件（以next主题为例）
`rm -R themes/next/.git* `
***

# 三、创建分支
## 1. 新建仓库
在blog项目仓库下，输入备份分支hexo，点击create创建（因为我已经创建过了，所以显示的不一样）
或者在本地使用命令 `git checkout -b hexo`
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705133032.png)
## 2. 点击设置，把默认分支设置为新建的备份分支
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705132053.png)
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705132123.png)
***

# 四、提交备份
在本地的hexo文件夹打开git bash，依次执行以下命令：

```
git add -all   #保存所有文件到暂存区
git commit -m "创建hexo分支" #提交变更
git push --set-upstream origin hexo
#推送到Github，并用`--set-upstream`与origin创建关联
#将hexo设置为默认分区
```
***

# 五、合并管理
将本地hexo分支中的.git文件夹复制到博客根目录中，
我们只需要手动管理hexo分支中的文件（备份），
.gitignore之外的文件由hexo管理（hexo d）
移除主题目录下的Git管理文件

`rm -R themes/next/.git* #以next主题为例`

master分支的文件则由hexo管理，编辑hexo配置文件*_config.yml*
```
deploy:
        type: git
        repo: https://github.com/yourusername/yourusername.github.io
        branch: master
```
***

# 六、发表文章及修改配置
## 1. 将相关更改（配置修改或发表文章）推送到hexo分支
```
git add .
git commit -m "修改配置/发表文章"
git push origin hexo
```
## 2. 将静态文件推送到master分支
```
hexo clean 
hexo g
hexo d
```
***

# 七、迁移
## 1. 环境安装
```
npm install -g hexo-cli
hexo init
npm install
```

## 2. 克隆hexo分支
`git clone -b hexo https://github.com/username/username.github.io`
***

这样就可以进行写作了，写完记得同步备份博客。
# 参考文章
[【GitHub】创建Git分支将Hexo博客迁移到其它电脑](https://blog.csdn.net/white_idiot/article/details/80685990)