---
title: 利用GitHub Pages+Hexo搭建个人博客（踩坑之路）
date: 2019-07-04 20:56:23
author: dylan
top: false
cover: false
password: 
categories: Blog
tags: 
        - Blog
        - Hexo
        - GitHub
---
***
# 前言
其实很早之前就想搭建一个个人博客，出于各种原因，一直没有行动。最近终于着手开始搭建，希望自己可以一直坚持下去。在搭建的过程中，踩了不少坑，特此记录，也希望对后来人有一点点参考价值。
***
# 一、Github
## 1. 注册Github账号
进入[Github](https://github.com)官网，注册账号。
## 2. 创建仓库
点击首页右上角头像左侧的头像
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705010509.png)
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705011051.png)
## 3. Github Pages
点击Settings
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705011609.png)
找到GitHub Pages，以用户名命名的仓库自动开启github pages，确认开启后就可以通过给出的网址访问了。
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705011514.png)
***

# 二、Hexo
[hexo](https://hexo.io/zh-cn/)是一个快速、简洁且高效的博客框架，可以参考官方文档。

## 1. 环境安装
要使用hexo，必须安装[Node.js](https://nodejs.org/en/download/)和[Git](https://git-scm.com/download/)。网上教程很多，这里不再赘述。

## 2. hexo安装
先创建存放blog文件的文件夹，切换到此文件夹右击git bash打开
输入命令安装hexo：
`npm install -g hexo-cli`

依次执行：
```
hexo init 
npm install
hexo g #生成静态网页
hexo s #启动本地服务
```
完成后，在浏览器输入localhost:4000就可以看到你的博客了
***

# 三、部署到Github
## 1. 设置SSH
返回GIt Bash中，依次输入：

```
git config --global user.name "yourname"
git config --global user.email "youremail"
```
这里的yourname输入你的Github的用户名，
youremail输入你的Github邮箱

```
cd ~/.ssh
ls
mkdir key_backup
cp id_rsa* key_backup
rm id_rsa*
#检查有没有生成过SSH并备份移除
ssh-keygen -t rsa -C "youremail" 
#生成新的SSH，接下来输入密码 一路回车
```
## 2.添加SSH Key到Github
点击头像，选择Settings
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705022259.png)
添加新的SSH Key
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705022426.png)
找到c:\users\当前用户名\.ssh    文本形式打开id_ras.pub (打开系统查看隐藏文件选项)
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190705022656.png)
测试是否连接成功
`ssh -T git@github.com`
输入yes就ok

## 3. 部署到Github
打开hexo配置文件（根目录）**_config.yml**
翻到最后将xxx修改为你的Github账户（冒号后面有一个空格）
```
deploy:
    type: git
    repo: https://github.com/xxx/xxx.github.io.git
    branch: master
```

然后安装deploy-git(不然报错”ERROR Deployer not found: git“)
`npm install hexo-deployer-git --save`

然后
```
hexo clean
hexo g
hexo d
```
deploy时可能要你输入密码，再刷新username.github.io就可以看到你的blog。

***
**注意：**
   >如果输入命令的过程中出现了"LF will be replaced by CRLF"报错，
    1. windows中的换行符为 CRLF，而在Linux下的换行符为LF，所以在执行add . 时出现提示 
    2. CRLF和LF是两种不同的换行格式，git工作区默认为CRLF来作为换行符，
        所以当我们项目文件里有用的地方使用LF作为换行符，这个时候我们再继续git add
        或者git commit的时候就会弹出警告，当最终push到远程仓库的时候git会统一格式全部转化为用CRLF作为换行符 

**解决办法：**

>1. 这个只是一个警告，我们直接忽略就好。
>2. git config –global core.autocrlf false  //禁用自动转换 
