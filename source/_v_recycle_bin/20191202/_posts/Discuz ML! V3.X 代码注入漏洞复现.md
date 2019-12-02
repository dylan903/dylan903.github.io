---
title: Discuz ML! V3.X 代码注入漏洞复现
date: 2019-07-22 14:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞复现
    - Discuz
---
# 一、漏洞概述
2019年7月11日， Discuz！ML被发现存在一处远程代码执行漏洞，
攻击者通过在请求流量的cookie字段中的language参数处插入构造的payload，
进行远程代码执行利用，该漏洞利用方式简单，危害性较大。
本次漏洞是由于Discuz! ML对于cookie字段的不恰当处理造成的
cookie字段中的language参数未经过滤，直接被拼接写入缓存文件之中，
而缓存文件随后又被加载，从而造成代码执行

**漏洞影响版本：**
Discuz!ML v.3.4 、Discuz!ML v.3.2 、Discuz!ML v.3.3 product of codersclub.org

***
# 二、漏洞复现
本地搭建Discuz！ML 环境
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190724113043.png)

在主页进行抓包，修改Language的值，添加  `'.phpinfo().'`
成功复现该漏洞
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190724114153.png)

***
# 三、漏洞修复
由于代码包含的原因，所以注入到缓存文件中的恶意代码直接执行，其中首页就有包涵，
全局搜索一下的话，应该有不少地方有进行包含可以直接利用，危害很大。
VulkeyChen师傅的建议：单看语言这个点，在/source/class/discuz/discuz_application.php 
第338行之后341行之前加入该代码暂缓此安全问题：
```
$lng = str_replace("(","",$lng);
$lng = str_replace(")","",$lng);
$lng = str_replace("'","",$lng);
$lng = str_replace('"',"",$lng);
$lng = str_replace('`',"",$lng);
```

***
# 参考文章
[Discuz ML! V3.X 代码注入漏洞深度分析](http://blog.topsec.com.cn/discuz-ml-v3-x-代码注入漏洞深度分析/)
[Discuz ML! V3.X 代码注入漏洞](https://www.cnblogs.com/-mo-/p/11180396.html)