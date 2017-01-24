---
title: 多git账号使用
date: 2017-01-25 04:22:26
tags:
        - git
        - mutli user
        - git config
---
相信不少同学公司在使用gitlab及其衍生产品，而另外自己也注册了Github账号，在这种情况下，
在一台机器环境下，如何做到能同时使用2种git?
1.添加配置区分不同域的git仓库
```
# gitlab2
Host gitlab2.dui88.com
User git
Hostname gitlab2.dui88.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa

# github
Host github.com
User git
Hostname github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_zzy_rsa
```
将内容保存为config文件后，保存到~/.ssh目录
将公司和GitHub的public key配置好
测试配置是否OK：
```
#测试github
ssh -T git@github.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.

#测试公司git
ssh -T git@gitlab2.dui88.com
Welcome to GitLab, xxx!
```
接下来就可以正常使用2个仓库了
不过，还有一点需要留意，因为我们在初始化git时，配置了user和email，是全局范围，这样两个仓库都会使用这个全局的配置
如果2个git的user，email是一样的也没什么问题，对于不同的情况，需要在仓库中写入本地配置
```
git config --local user.name 'xxx'
git config --local user.email 'xxx@compary.com'
```