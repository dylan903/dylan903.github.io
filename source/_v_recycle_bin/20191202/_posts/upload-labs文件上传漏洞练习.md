---
title: upload-labs文件上传漏洞练习
date: 2019-08-4 13:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 靶场
    - dcoker
    - upload-labs
    - 文件上传
---
# 0x01 前言
最近在研究文件上传漏洞，找到一个很好的靶场——[upload-labs](https://github.com/c0ny1/upload-labs)，
一个想帮你总结所有类型的上传漏洞的靶场 ，可以用docker快速搭建，闯关的过程中遇到很多问题，受益匪浅。

***
# 0x02 环境搭建
使用docker快速搭建，docker的安装这里不再赘述。

`docker pull c0ny1/upload-labs`
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804145816.png)

创建容器
`docker run -d -p 8000:80 c0ny1/upload-labs:latest`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804150235.png)

本地环境，访问`127.0.0.1:8000`
云服务器上搭建的，访问`服务器ip:8000`（注意开放防火墙端口，阿里云服务器需要在云控制台配置开放端口）

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804150626.png)

***
# 0x03 闯关
## Pass-01（前端）
这一关是在客户端使用js对不合法图片进行检查，直接F12，把调用相关js的代码删掉，直接上传拿shell

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804151107.png)

## Pass-02（MIME）
第二关主要是检查MIME，直接抓包修改Content-Type（例如：`image/gif`）上传即可

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804151431.png)

## Pass-03（特殊可解析后缀）
第三关是黑名单禁止上传.asp|.aspx|.php|.jsp后缀文件，尝试另类文件名绕过。（phtml，php3，php4, php5, pht等）
直接抓包修改文件后缀，将php改为phtml，php3，php4, php5, pht等，上传成功
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804152622.png)

但是很鸡肋，如果服务器没有配置别名解析，上传上去是无法被解析执行的。
如果无法解析执行，需要修改apache配置文件。
这里以docker搭建的环境为例：
输入命令进入容器内部：
```
docker exec -it condescending_nightingale /bin/bash 
# 这里的condescending_nightingale是容器的name，可以输入docker ps查看
```
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804153104.png)

```
vim /etc/apache2/apache2.conf
```
修改apache2配置文件，添加下面这句话
`AddType application/x-httpd-php .php .phtml .phps .php3 .php5 .pht`

退出容器内部
`exit`

重启容器，即可解析成功
`docker restart condescending_nightingale`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804154224.png)

## Pass-04（htaccess）
这一关过滤了各种罕见后缀，但是没有过滤`.htaccess`文件
.htaccess文件(或者”分布式配置文件”）,全称是Hypertext Access(超文本入口)。提供了针对目录改变配置的方法，
即在一个特定的文档目录中放置一个包含一个或多个指令的文件， 以作用于此目录及其所有子目录。
作为用户，所能使用的命令受到限制。管理员可以通过Apache的AllowOverride指令来设置。
启用.htaccess，需要修改`apache2.conf`**(同Pass-03)**，启用AllowOverride.

`AllowOverride None` 
改为 
`AllowOverride All`

如果需要使用.htaccess以外的其他文件名，可以用AccessFileName指令来改变。
例如，需要使用.config ，则可以在服务器配置文件中按以下方法配置：

`AccessFileName .config ` 

然后执行命令启用Mod_rewrite模块

`sudo a2enmod rewrite`

最后重启apache2

`service apache2 restart`

使用 快捷键`ctrl + p + q `退出容器（不会中止容器）

先上传`.htaccess`文件，文件内容如下(引号内替换成你要上传执行的文件名)：
```
    <FilesMatch "cmd.jpeg">
      SetHandler application/x-httpd-php
    </FilesMatch>
```

windows系统文件不能命名为`.*`，所以在上传的时候抓包，改文件名，删掉点前面的

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804215949.png)

然后上传图片木马文件  `cmd.jpeg` ，成功解析。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190804220610.png)

## Pass-05（大小写）
这一关把`.htaccess`后缀也禁止了，查看源代码，发现把转换成小写的代码去掉了

` $file_ext = strtolower($file_ext); //转换为小写`

因此我们可以上传Php、phP之类的来绕过黑名单后缀，成功上传。
(在Linux没有特殊配置的情况下，这种情况只有win可以解析执行，因为win会忽略大小写)

## Pass-06（空格）
这一关，少了这一段代码
` $file_ext = trim($file_ext); //首尾去空`

可以进行空格绕过，直接抓包修改文件名，再文件名末尾添加空格，成功上传

## Pass-07（点）
这一关少了这段代码

`$file_name = deldot($file_name);//删除文件名末尾的点`

没有对后缀名进行去”.”处理，利用windows特性，会自动去掉后缀名中最后的”.”，可在后缀名中加”.”绕过。

## Pass-08（::$DATA）

`$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA`
NTFS文件系统包括对备用数据流的支持，主要包括提供与Macintosh文件系统中的文件的兼容性。
备用数据流允许文件包含多个数据流。每个文件至少有一个数据流。在Windows中，此默认数据流称为：`$ DATA`。
上传.php::$DATA绕过。(仅限windows)

## Pass-09(代码审计)
```
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
```
查看源码，这里只过滤了一次，所以直接构造 `.php. .` 绕过

## Pass-10(双写)

` $file_name = str_ireplace($deny_ext,"", $file_name);`

这里是将黑名单里的后缀替换为空，可以利用双写绕过
构造`.pphpph`,成功上传解析执行。

## Pass-11（%00截断）

`$img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;`

可以参考这篇文章：[PHP任意文件上传漏洞CVE-2015-2348浅析](https://www.cnblogs.com/cyjaysun/p/4390930.html)

`save_path` 是一个可控的变量，可以使用%00截断
使用条件：

* php 版本<5.3.4 才有可能存在此漏洞
* php的magic_quotes_gpc为OFF状态

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805093837.png)

## Pass-12（0x00截断）

原理同Pass-11,只不过`save_path`是通过post传进来的，需要在Hex里修改

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805100157.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805095929.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805100023.png)

`+`的URL编码的16进制 为2b，将2b改为00即可

## Pass-13（文件头）
这一关通过读文件的前2个字节判断文件类型
直接使用 cmd命令生成图片木马上传

`copy pikachu.gif /b + cmd.php /a cmd.gif`

用给出的文件包含漏洞页面来测试图片马是否能正常运行！ 
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805103257.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805103451.png)

## Pass-14（getimagesize）

这一关使用`getimagesize`获取文件类型，直接就可以利用图片马进行绕过。（同Pass-13）

## Pass-15（exif_imagetype()）

本关使用`exif_imagetype()`检查是否为图片文件,直接就可以利用图片马就可进行绕过。

## Pass-16（二次渲染）

本关判断了后缀名、content-type，以及利用imagecreatefromgif判断是否为gif图片，最后再做了一次二次渲染
具体可以参考这篇文章：[upload-labs之pass 16详细分析](https://xz.aliyun.com/t/2657)

先上传图片码，然后下载下来，用16进制编辑器打开，寻找图片被渲染后与原始图片部分对比仍然相同的数据块部分，
将Webshell代码插在该部分，然后上传即可
jpg和png很麻烦，gif直接修改没有改变的区域即可。

## Pass-17（条件竞争）

本关文件先经过保存，然后判断后缀名是否在白名单中，如果不在则删除。
此时可以利用条件竞争在保存文件后删除文件前来执行php文件。
可以用burp suite中的Intruder模块同时批量上传、访问webshell，
将payloads中的payload type设置为Null payload，
Generate payload次数多点。

## Pass-18（条件竞争）

和Pass-17一样，也是一个条件竞争的问题，查看源代码
对文件后缀名做了白名单判断，然后会一步一步检查文件大小、文件是否存在等等，将文件上传后，对文件重新命名等。
可以不断利用burp发送上传图片马的数据包，由于条件竞争，程序会出现来不及rename的问题，从而上传成功

## Pass-19（代码审计）

```
$img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传失败！';
            }
```
`move_uploaded_file()`函数中的`img_path`是由post参数`save_name`控制的，因此可以在`save_name`利用00截断绕过。

另外**`move_uploaded_file`**会忽略掉文件末尾的 `/.`
所以可以构造 `cmd.php/.` 来绕过
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805121815.png)

## Pass-20（代码审计）
这个题目用了数组+/.的方式去绕过，因为源代码里面含有这样的两句代码，成了关键得绕过的地方

```
if (!is_array($file)) {
                    $file = explode('.', strtolower($file));
                }
```
```
$file_name = reset($file) . '.' . $file[count($file) - 1];
```

这同样我们就需要满足两个条件，第一个是先得保证另外修改的名字需要满足是数组的条件，所以我们可以抓包构造数组，
第二点由于后面filename构成的过程中由于`$file[count($file) - 1]`的作用，导致`$file[1] = NULL`，所以构造文件名后相当于直接就是`xx.php/.`，
根据上面一题的知识，可以直接在`move_uploaded_file`函数的作用下可以将/.忽略，因此还是可以上传成功的。
因此`save_name`变量的两个值分别是`xx.php/`，另外一个值是`jpg`，其实从代码审计的角度上看，还是可控变量导致这样的后果

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805123452.png)

***
# 0x04 总结
## upload-labs总结
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805134449.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190805134922.png)

其中有一些漏洞因为环境原因，没能成功解析执行。
顺便在总结一下其他中间件问题导致的解析漏洞

## IIS 6.0
IIS 6.0解析利用方法有三种：
1.目录解析
建立xx.asp为名称的文件夹，将asp文件放入，访问/xx.asp/xx.jpg，其中xx.jpg可以为任意文件后缀，即可解析
2.文件解析
后缀解析：/xx.asp;.jpg /xx.asp:.jpg(此处需抓包修改文件名)
3.默认解析
IIS6.0 默认的可执行文件除了asp还包含这三种
```
/xxx.asa
/xxx.cer
/xxx.cdx
/xxx.apsx
```

## IIS 7.0/7.5
在正常图片URL后添加 /.php，可以解析为php

## Apache
一般都在2.3.x以下版本，但是有时候配置文件的不同也会导致不安全

后缀解析：test.php.x1.x2.x3
Apache将从右至左开始判断后缀，若x3非可识别后缀，再判断x2，直到找到可识别后缀为止，然后将该可识别后缀进解析
test.php.x1.x2.x3则会被解析为php

apache 2.1.x的版本就可以用test.php.jpg直接就可以getshell了

## Nginx
Nginx <8.03畸形解析漏洞
直接在正常图片URL后添加/.php
Nginx <=0.8.37
在Fast-CGI关闭的情况下，Nginx <=0.8.37 依然存在解析漏洞

在一个文件路径(/xx.jpg)后面加上%00.php会将 /xx.jpg%00.php 解析为 php 文件。

***
# 参考文章
[Upload-labs 20关通关笔记](https://xz.aliyun.com/t/4029)
[upload-labs刷关记录](https://blog.csdn.net/u011377996/article/details/86776198)
[从upload-labs总结上传漏洞及其绕过 ](http://poetichacker.com/writeup/从upload-labs总结上传漏洞及其绕过.html)
[文件解析漏洞总结](https://www.smi1e.top/文件解析漏洞总结/)