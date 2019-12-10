---
title: Cobalt Strike使用指南—入门篇
date: 2019-12-9 14:54:16
author: dylan
top: false
cover: false
summary: Cobalt Strike使用指南
categories: Tools
tags: 
    - Cobalt Strike
    - CS
    - 远控
    - 内网渗透
---
# 0x00 CS简介

Cobalt Strike 一款以Metasploit为基础的GUI框架式渗透测试工具，集成了端口转发、服务扫描，自动化溢出，
多模式端口监听，exe、powershell木马生成等。

Cobalt Strike 分为客户端和服务端，可分布式操作、协同作战。
但一定要架设在外网上，或者自己想要搭建的环境中。

# 0x01 部署安装
## 环境

Team Server 推荐运行环境

* Kali Linux 1.0, 2.0 – i386 and AMD64
* Ubuntu Linux 12.04, 14.04 – x86, and x86_64

Client GUI 运行环境

* Windows 7 and above
* macOS X 10.10 and above
* Kali Linux 1.0, 2.0 – i386 and AMD64
* Ubuntu Linux 12.04, 14.04 – x86, and x86_64

由于`cobaltstrike`为`java`编写,
所以客户端和服务端都需要安装`java`运行环境

## 服务端

服务端关键的文件是`teamserver`以及`cobaltstrike.jar`
将这两个文件放到服务器上同一个目录，然后运行：

`chmod 755 teamserver`
`sudo ./teamserver 192.168.1.1 asdf # ip 和密码`

如果是本地测试，本机既当服务端，又当客户端
可填入本机ip，不要填`127.0.0.1`或者`0.0.0.0`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191209153500.png)


乌班图如果出现这个报错。

```
[-] keytool is not in $PATH
    install the Java Developer Kit
```

加一个软链即可

`ln -s /usr/java/jdk/bin/keytool /usr/bin/`

## 客户端

windows客户端打开`cobaltstrike.bat`批处理文件

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191209153619.png)

输入服务端的设置的IP、端口、密码，用户名随意，登陆即可

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191209153902.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191209154047.png)


# 0x02 简单使用

这里只介绍下简单操作，其它后面学习了再总结。

## 设置Listeners

这里使用`win7`虚拟机进行演示

首先添加`Listeners`,填入服务端的IP，和想要监听的端口

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210171137.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210171355.png)


## 生成木马

生成exe木马

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210171522.png)

根据目标来选择是否使用`x64`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210171601.png)

将生成的exe文件复制到虚拟机双击执行
即可看到靶机上线

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210171942.png)

## 执行命令

在Cobalt Strike中它的心跳默认是60s
即sleep时间为60s，每一分钟目标主机与teamserver通信一次，
所有执行命令或者其它操作都会等待60s，所有先执行修改sleep为0
(根据实际情况修改，太快流量明显)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210172025.png)

右击，选择`Interact`,最下面可以输入命令
beacon中不能直接输入cmd命令，如果要执行cmd命令，前面要加上`shell` 
例如：`shell whoami`
其他的`beacon`命令，大家可以在`beacon`中输入`help`查看

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191210172627.png)



三方提权模块：
[ElevateKit: The Elevate Kit demonstrates how to use third-party privilege escalation attacks with Cobalt Strike's Beacon payload.](https://github.com/dylan903/ElevateKit)

点击Script Manager按钮打开脚本管理界面，然后点击底下的Load按钮加载elevate.cna文件
如果提权成功，session列表中会增加一个新会话，星号(*)表示该会话是一个提权成功的会话。
提权成功后我们再执行危险操作时被控PC就不会有UAC提示，被控目标也不会察觉。



# 参考文章

[Cobalt Strike使用 - print("")](https://www.o2oxy.cn/2361.html)
[渗透利器Cobalt Strike - 第1篇 功能及使用](https://xz.aliyun.com/t/3975)
[Cobalt Strike学习笔记（持续更新）](https://www.freebuf.com/sectool/133369.html)
[Cobaltstrike系列教程(一)简介与安装=](https://blog.csdn.net/qq_26091745/article/details/98097401)