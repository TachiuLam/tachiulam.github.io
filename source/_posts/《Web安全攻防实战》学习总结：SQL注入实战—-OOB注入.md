---
title: 《Web攻防安全实战》学习总结：SQL注入实战—-OOB注入
date: 2021-03-27 15:02:24
index_img: /img/websecurity.png
banner_img: /img/websecurity.png
tags:
  - 安全
  - Web安全
categories:
  - Web安全
  - SQL注入
---
# OOB注入涉及原理
## OOB注入（Out of Band：
带外通道技术（OOB）让攻击者能够通过另一种方式来确认和利用没有直接回显的漏洞。这类漏洞中，攻击者无法通过恶意请求直接在响应包中看到漏洞的输出结果。
带外通道技术通常需要脆弱的实体来生成带外的TCP/UDP/ICMP请求，然后攻击者可以通过这个请求提取数据。
## 泛域名解析
泛域名解析就是利用通配符的方式将所有的次级域名指向同一IP。如www.example.com和abc.example.com都会访问到同一个站点。
## UNC路径：
Universal Naming Convention/通用命名规则。
Windows主机默认存在，Linux主机默认不存在。格式：\\servername\sharename，其中servername是服务器名，sharename是共享资源的名称。
我们平时使用打印机、网络共享文件夹时，都会用到UNC填写地址，并且在当我们在使用UNC路径时，会对域名进行DNS查询。
# 环境搭建
## CEYE平台
CEYE平台可以监控DNS请求，并且配置了泛域名解析。
![fileupload](/img/sql/oob/ceye1.png)
对注册后分配到域名和其泛域名进行解析测试，可在平台上成功查询到解析记录：
![fileupload](/img/sql/oob/ceye2.png)
![fileupload](/img/sql/oob/ceye3.png)
## phpstudy
安装phpstudy，启动Apache、Mysql服务器;同时安装较低版本的php，如5.2.17：
![fileupload](/img/sql/oob/phpstudy1.png)
![fileupload](/img/sql/oob/phpstudy2.png)
## MySQL
修改MySQL服务器权限，运行root用户远程登录：
![fileupload](/img/sql/oob/phpstudy3.png)

