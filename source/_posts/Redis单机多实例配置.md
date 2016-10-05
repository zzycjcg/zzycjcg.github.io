---
title: Redis单机多实例配置
date: 2016-09-29 22:52:50
tags: 
        - redis 
        - sentinel
---

apt-get install redis-server安装redis后，默认是单点redis，为了研究redis集群，决定在单机上配置多个redis实例。
所谓redis实例，就是指多个redis进程，它们分别监听多个端口，单独都可以提供服务。

<!-- more -->

## 拷贝文件配置并修改
安装好redis之后，默认会在/etc/redis下存放2个配置文件：redis-server.conf、sentinel.conf，分别是redis服务端配置和redis哨兵配置。
使用已经存在的6379端口示例配置，拷贝一份新的文件
```
    user@ubuntu:/etc/redis$ sudo cp redis.conf 6380.conf
    vi 6380.conf
    #修改pidfile
    pidfile /var/run/redis/redis-6380.pid
    #修改port
    port 6380
    #修改bind (O)
    bind 0.0.0.0
    #修改logfile
    logfile /var/log/redis/redis-6380.log
    #修改dump file name
    dbfilename dump-6380.rdb
```
最后，将默认的redis-server.conf也修改为redis-6379.conf，使命名风格统一。在后面会看到，配置文件命名风格统一后可以很方便统一配置自启动脚本。
```
    sudo mv redis-server.conf redis-6379.conf
```
同时也修改pidfile，logfile，dumpname

## 完成conf关联的修改
这一步是可选的，因为如果不配置，redis在启动时会帮我们自动创建。
```
    #pidfile
    cd /var/run/redis/
    touch redis-6380.pid
    #log file
    cd /var/log/redis/
    sudo touch redis-6380.log
```
## redis自启动脚本修改
### 修改6379实例配置
```
    #进入自动启动脚本目录
    cd /etc/init.d
    #找到redis启动脚本
    ll | grep redis
```
默认为看到一个redis-server的脚本，修改redis-server名称，即redis-6379
```    
    mv redis-server redis-6379
    vi redis-6379
```
将脚本前面的变量定义修改为如下：
``` 
    #定义redis实例名称
    REDIS_INSTANCE="redis-6379"
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
    DAEMON=/usr/bin/redis-server
    #引用redis实例变量
    DAEMON_ARGS=/etc/redis/${REDIS_INSTANCE}.conf
    NAME=${REDIS_INSTANCE}
    DESC=${REDIS_INSTANCE}

    RUNDIR=/var/run/redis
    PIDFILE=$RUNDIR/${REDIS_INSTANCE}.pid
```
### 增加6380实例配置
```
    #拷贝一份6379脚本
    cp redis-6379 redis-6380
    #修改实例名称
    REDIS_INSTANCE="redis-6380"
```
### 更新自启动
```
    #先删掉旧的redis-server启动
    update-rc.d –f redis-server remove
    #添加6379自启动
    update-rc.d redis-6379 defaults
    #添加6380自启动
    update-rc.d redis-6380 defaults
```
## 验证
```
    #分别启动6379和6380实例
    service redis-6379 start
    service redis-6380 start
    #查询启动的redis进程
    ps -ef | grep redis-server | grep -v grep
```
![Redis进程](/images/redis-pid-1.png)
```
    #查看redis信息
    redis-cli –h 192.168.1.111 –p 6379
    192.168.1.111:6379>info replication
```
```
    #reboot验证看Ubuntu启动时，2个redis实例是否自动启动
```
## 问题

总体来说，配置还是很简单的，但才开始玩redis，很是不熟，google了不少blog学习，redis实例本身配置这块是很简单的。主要的问题在启动脚本：

* 问题1：自启动脚本启动后无法启动。
开始参照一篇blog[1]，给、/etc/init.d/redis-6379建软链接，去掉第一行的变量定义，改为通过获取启动命令获取实例名称。这样通过service启动成功，但是重启后怎么也启动不了，一开始怀疑是添加自启动未生效，还弄了好半天，结果无解；最后修改变量获取方法，定义实例常量值，结果成功了。由此得出结论，Ubuntu启动时，调用启动命令方式不是service xxx start
    #通过service启动，用basename可以把redis-6379读取作为实例名称
    service redis-6379 start

* 问题2：service redis-6379 status无效
需要跟踪status问题原因并解决。
    #service redis-6379 status查看状态永远是not running
    service redis-6379 status
    ![Redis状态](/images/redis-status-1.png)

* 遗留问题：
    1. Ubuntu系统启动时，是如何来调用启动脚本来启动需要自启动的应用的？
    2. 是否可以优化脚本，在一个启动脚本里面循环拉起多个redis启动？
    3. service redis-6379 status命名问题出在哪？如何修改？

## 参考
    
* [How-to-setup-and-run-mulitple-Redis-server-instances-on-a-Linux-host](https://support.pivotal.io/hc/en-us/articles/206087627-How-to-setup-and-run-mulitple-Redis-server-instances-on-a-Linux-host)


