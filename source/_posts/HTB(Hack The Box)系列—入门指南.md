---
title: HTB(Hack The Box)系列—入门指南
date: 2019-12-2 13:14:52
author: dylan
top: false
cover: false
categories: HTB系列
tags: 
    - HTB
    - Hack The Box
    - 靶场
---

# 0x00 前言


# 0x01 注册登陆

注册需要邀请码

## 快速获取邀请码
### 方法一

首先打开邀请界面

[https://www.hackthebox.eu/invite](https://www.hackthebox.eu/invite)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203105144.png)

按下F12，打开`Console`
输入如下代码执行

```
$.post('https://www.hackthebox.eu/api/invite/generate',function(data){console.log(data)})`
// 这里的$就是jQuery不是Chrome里querySelector，因为这个页面已经载入了jQuery。
```

会返回如下数据：

`{"success":1,"data":{"code":"TExRRlAtSldNTkstVEdNWFotU1pIQVItRlhWWkM=","format":"encoded"},"0":200}`

`code`的值即为加密的邀请码

`TExRRlAtSldNTkstVEdNWFotU1pIQVItRlhWWkM=`

进行`Base64`解密即可，注意要把复制的双引号去掉

`LLQFP-JWMNK-TGMXZ-SZHAR-FXVZC`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203112444.png)

### 方法二

使用curl或者其它任何可以发送POST请求的工具都可以
得到返回数据解密即可

```
curl -X POST https://www.hackthebox.eu/api/invite/generate
{"success":1,"data":{"code":"RkxFR0wtV1VEWk4tRUhQRkMtR1RRVlMtT0ZIWFk=","format":"encoded"},"0":200}
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203113630.png)


## 详细步骤

首先打开邀请界面

[https://www.hackthebox.eu/invite](https://www.hackthebox.eu/invite)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203105144.png)

按下F12,打开Network

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203110206.png)

有一个`inviteapi.min.js`的请求（没有的话就刷新下）
直译就是邀请API，双击打开

[https://www.hackthebox.eu/js/inviteapi.min.js](https://www.hackthebox.eu/js/inviteapi.min.js)

内容如下：

```
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

加过密的JS代码，直接百度js解密
解密之后的代码如下：

```
function verifyInviteCode(code) {
    var formData = {
        "code": code
    };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/invite/verify',
        success: function(response) {
            console.log(response)
        },
        error: function(response) {
            console.log(response)
        }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/invite/how/to/generate',
        success: function(response) {
            console.log(response)
        },
        error: function(response) {
            console.log(response)
        }
    })
}
```

定义了两个函数，`verifyInviteCode(code)`和`makeInviteCode()`
这里直接在邀请页面`F12`，打开控制台`Console`，执行`makeInviteCode()`
返回如下内容：

`{0: 200, success: 1, data: {…}}`

点击三角符号展开，会发现这里的data部分是被加密的。
返回值中包含了加密的文本`data`和加密方式`enctype`，如下图加密方式为`"BASE64"`
加密方式是随机的几种,网上搜索对应的解密工具即可,解密后得到下面的内容。

`In order to generate the invite code, make a POST request to /api/invite/generate`

然后安装上面的快速获取邀请码的步骤执行即可。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203111338.png)


# 0x02 kali连接HTB

在`access`页面点击`Connection Pack`下载`openvpn`配置文件
或者点击如下连接下载

[https://www.hackthebox.eu/home/htb/access/ovpnfile](https://www.hackthebox.eu/home/htb/access/ovpnfile)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203114624.png)


`openvpn`配置文件复制到`kali`，并在所在路径执行

`sudo openvpn --config xxxx.ovpn`

`openvpn`kali最新版好像有，如果没有安装一下再执行就好了。

`sudo apt-get install openvpn`

执行成功后就会打勾,执行成功后不用关闭终端。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203115315.png)

# 参考文章

[Hack The Box 获取邀请码](https://www.cnblogs.com/zhuxiaoxi/p/10305940.html)