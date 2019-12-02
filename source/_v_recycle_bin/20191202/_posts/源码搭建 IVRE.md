---
title: Ubuntu16 源码搭建 IVRE
date: 2019-09-21 13:22:42
author: dylan
top: false
cover: false
password: 
categories: summary
tags: 
    - Ubuntu16
    - 源码
    - IVRE
    - tools
    - 网络侦查框架
---
# 0x01 前言
IVRE(又名DRUNK)是一款开源的网络侦查框架工具，IVRE使用Nmap、Zmap进行主动网络探测，
使用Bro、P0f等进行网络流量被动分析，探测结果存入数据库中，方便数据的查询、分类汇总统计。
网上大多都是使用Docker进行安装，配置便捷，不会遇到太多问题，但有时候特殊需求需要用源码
进行安装，特此记录一下。
可以参考官方文档：[Overview — IVRE documentation](https://doc.ivre.rocks/en/latest/overview/index.html)
Github：[ivre: Network recon framework.](https://github.com/cea-sec/ivre)

# 0x02 Install&Setup
## Install
安装需要的服务器数据库
`sudo apt-get -y install mongodb python-pymongo python-crypto \`
`python-future python-bottle apache2 libapache2-mod-wsgi dokuwiki`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190921161300.png)

下载源码
`git clone https://github.com/cea-sec/ivre`

源码安装
（推荐用Python2，Python3踩坑见文末）
`cd ivre`
`python3 setup.py build`
`sudo python3 setup.py install`

## Setup
```
$ sudo -s
# cd /var/www/html ## or depending on your version /var/www
# rm index.html
# ln -s /usr/local/share/ivre/web/static/* .
# cd /var/lib/dokuwiki/data/pages
# ln -s /usr/local/share/ivre/dokuwiki/doc
# cd /var/lib/dokuwiki/data/media
# ln -s /usr/local/share/ivre/dokuwiki/media/logo.png
# ln -s /usr/local/share/ivre/dokuwiki/media/doc
# cd /usr/share/dokuwiki
# patch -p0 < /usr/local/share/ivre/dokuwiki/backlinks.patch
# cd /etc/apache2/mods-enabled
# for m in rewrite.load wsgi.conf wsgi.load ; do
>   [ -L $m ] || ln -s ../mods-available/$m ; done
# cd ../
# echo 'Alias /cgi "/usr/local/share/ivre/web/wsgi/app.wsgi"' > conf-enabled/ivre.conf
# echo '<Location /cgi>' >> conf-enabled/ivre.conf
# echo 'SetHandler wsgi-script' >> conf-enabled/ivre.conf
# echo 'Options +ExecCGI' >> conf-enabled/ivre.conf
# echo 'Require all granted' >> conf-enabled/ivre.conf
# echo '</Location>' >> conf-enabled/ivre.conf
# sed -i 's/^\(\s*\)#Rewrite/\1Rewrite/' /etc/dokuwiki/apache.conf
# echo 'WEB_GET_NOTEPAD_PAGES = "localdokuwiki"' >> /etc/ivre.conf
# service apache2 reload  ## or start
# exit
```

完成后，访问[http://localhost/](http://localhost/),服务器访问ip，即可看到WEB UI

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190921163114.png)

如果搭建在服务器上，记得点击help，查看是否正常工作。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190921164623.png)

如果显示 `Forbidden`,使用如下命令修改配置文件
`vim /etc/dokuwiki/apache.conf`
将
`Allow from localhost 127.0.0.1 ::1`
修改为：
```
#Allow from localhost 127.0.0.1 ::1
Allow from all
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190921165037.png)

# 0x03 Database init, data download & importation
这一步时间有点长，耐心等待

```
$ yes | ivre ipinfo --init
$ yes | ivre scancli --init
$ yes | ivre view --init
$ yes | ivre flowcli --init
$ yes | sudo ivre runscansagentdb --init
$ sudo ivre ipdata --download --import-all
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190921203152.png)

# 0x04 Run a first scan
## Against 1k (routable) IP addresses, with a single nmap process:

`sudo ivre runscans --routable --limit 200`

如果报如下错误：
```
nmap: unrecognized option '--script-timeout'
ADDING TARGET 1 : 120.75.39.145
ERROR: NMAP PROCESS IS DEAD
```
或者
```
Traceback (most recent call last):
  File "/usr/local/bin/ivre", line 84, in <module>
    main()
  File "/usr/local/bin/ivre", line 56, in main
    tools.get_command(next(iter(possible_commands)))()
  File "/usr/local/lib/python2.7/dist-packages/ivre/tools/runscans.py", line 498, in main
    accept_target_status=accept_target_status)
  File "/usr/local/lib/python2.7/dist-packages/ivre/tools/runscans.py", line 217, in call_nmap
    stdin=subprocess.PIPE, stdout=subprocess.PIPE)
  File "/usr/lib/python2.7/subprocess.py", line 711, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1343, in _execute_child
    raise child_exception
OSError: [Errno 2] No such file or directory
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190920111226.png)

因为 nmap未安装或者版本过低，Nmap Debian 版本可能比当前的版本晚一年甚至更长的时间
下载最新 RPM 格式的 nmap 包，然后使用 alien 工具把他转换成 debian 包，再用 dpkg 工具安装即可。

步骤如下：

* 先卸载已安装的nmap   
 `sudo apt remove nmap -y `
* 安装 alien  
 `sudo apt-get install alien`
* 使用wget下载  [Nmap RPMs](https://nmap.org/download.html)     
 `wget https://nmap.org/dist/nmap-7.80-1.x86_64.rpm`
* 转化
 `sudo alien nmap-7.80-1.x86_64.rpm`
* 安装 
 `sudo dpkg --install nmap_7.80-2_amd64.deb`

## import the results and create a view

```
$ ivre scan2db -c ROUTABLE,ROUTABLE-CAMPAIGN-001 -s MySource -r \
>              scans/ROUTABLE/up
$ ivre db2view nmap
```
返回Web UI，刷新即可查看扫描结果。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190921213155.png)


## WEB-UI 没有数据

安装好后，数据导入了数据库，web-ui界面一直显示Conuting，没有数据

`cat /var/log/apache2/error.log`

查看apache错误日志，发现
使用的是Python3 安装的，但是它会调用Python2，所以报错
解决办法是

直接卸载python2，
或者
使用python2来安装IVRE。

如果还不出来，再查看apache错误日志，根据报错来改，可能缺少模块。

# 参考文章
[IVRE! Drunk Frenchman Port Scanner Framework!](https://mstajbakhsh.ir/ivre-drunk-frenchman-port-scanner-framework/)
[Overview — IVRE documentation](https://doc.ivre.rocks/en/latest/overview/index.html)