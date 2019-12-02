---
title: zzzphp V1.6.1 远程代码执行漏洞复现
date: 2019-07-24 14:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞
    - 复现
    - zzzphp
---
# 一、漏洞概述

远程代码执行漏洞存在的主要原因是页面对模块的php代码过滤不严谨，
导致在后台可以写入php代码从而造成代码执行。

***
# 二、漏洞复现

本地搭建zzzphp V1.6.1环境
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190724141219.png)

在后台模块管理中的电脑模块找到cn2016
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190724141335.png)

然后在cn2016文件中到html文件，然后在html文件中找到search.html，然后将其的代码修改为
`{if:assert($_request[phpinfo()])}phpinfo();{end if}`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190724141523.png)

然后打开`http://xxx/zzzcms/search/`就可以看到我们刚刚输入的phpinfo()执行了。
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190724141748.png)

***
# 参考文章
[zzzphp V1.6.1 远程代码执行漏洞分析](https://xz.aliyun.com/t/4471)
[zzzphpV1.6.1 远程代码执行漏洞简单分析](https://www.anquanke.com/post/id/173991)