---
title: WinRAR目录穿越漏洞复现
date: 2019-07-26 14:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞
    - 复现
    - WinRAR
    - 目录穿越
---
# 0x01 漏洞概述
该漏洞是由于 WinRAR 所使用的一个陈旧的动态链接库UNACEV2.dll所造成的，该动态链接库在 2006 年被编译，
没有任何的基础保护机制(ASLR, DEP 等)。动态链接库的作用是处理 ACE 格式文件。
而WinRAR解压ACE文件时，由于没有对文件名进行充分过滤，导致其可实现目录穿越，
将恶意文件写入任意目录,甚至可以写入文件至开机启动项，导致代码执行。

***
# 0x02 漏洞影响
**影响版本：**
       * WinRAR < 5.70 Beta 1
       * Bandizip < = 6.2.0.0
       * 好压(2345压缩) < = 5.9.8.10907
       * 360压缩 < = 4.0.0.1170
       * ……

***
# 0x03 漏洞复现
新建一个任意文件，名称类型内容随意
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726155542.png)

使用Winace进行压缩
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726155906.png)

然后下载[acefile.py](https://github.com/droe/acefile/blob/master/acefile.py)脚本
输入命令`python acefile.py --headers test.ace` 读取文件的头部信息
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726160308.png)


ok，开始构造恶意文件
用 010Editor 打开test.ace文件
需要修改以下参数：
* hdr_crc
* hdr_size
* filename的长度
* filename
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726160517.png)
首先将filename的值改为 `d:\d:\liehu.txt`
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726161040.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726161237.png)

修改后的filename的长度，选中它，左下角就是它的长度15，16进制为00 0F，filename的前两位就是它的长度
修改顺序是由后到前，即将10改为0F即可
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726161841.png)

修改**hdr_size**，选中如下位置，左下角查看其长度，这里是（00 2E），选中的前面的红框就是hdr_size
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726162147.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726162439.png)

最后修改**hdr_crc**，再次运行
`python acefile.py --headers test.ace`  
CRC校验失败，报错
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726162831.png)
在acefile.py文件中查找 `header CRC failed`
在其上面一行添加输出语句，输出ace_crc16(buf)，即为我们需要的**hdr_crc**的值
`print (ace_crc16(buf), buf)`
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726163206.png)

改好保存，再次运行
`python acefile.py --headers test.ace`  
31102即我们需要的值，转换成16进制为79 7E
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726163625.png)

将**hdr_size** 前面的两位即为**hdr_crc**，从右到左修改为79 7E
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726164206.png)

再次运行
`python acefile.py --headers test.ace`  
输出如下信息无报错，就成功了，用开头所述解压工具解压test.ace，就会在红框的路径生成对应的文件。
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726164506.png)

***
# 0x04 修复建议
1. 升级最新的WinRAR ，目前版本是 5.71 
2. winRAR安装目录下，删除UNACEV2.dll文件

***
# 参考文章
[WinRAR漏洞复现过程](https://fuping.site/2019/02/21/WinRAR-Extracting-Code-Execution-Validate/)
[Extracting a 19 Year Old Code Execution from WinRAR - Check Point Research](https://research.checkpoint.com/extracting-code-execution-from-winrar/)
