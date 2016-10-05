---
title: root无法使用ssh或sftp的问题
date: 2016-10-05 11:25:44
tags: sshd
---
默认情况下，安装好ssh后，是不能用root登陆使用的，google了一把，原来是被Ubuntu禁用了。<br>
直接上配置：

<!-- more -->

```
#修改/etc/ssh/sshd_config
vi /etc/ssh/sshd_config
# allow root ssh login: 1.comment 'without-password'; 2.add yes
#PermitRootLogin without-password
PermitRootLogin yes
#重启ssh服务
service ssh restart
```
