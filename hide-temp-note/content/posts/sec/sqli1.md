---
title: "MySQL 联合注入"
date: 2024-02-01
description: "靶场演示"
tags: ["SQL 注入", "后端漏洞" ]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

对 mysql 进行联合注入是在有回显点的情况下用 union select 直接查询数据库内容，查询结果在回显点用 group_concat 输出。常见的关系型数据库都有系统自带库，mysql 的是 information_schema 这个库，其中存储着全部库名、表名、字段名。我们这里要做的是在 information_schema 找到关键的库、表、字段，从而找到 flag。

靶场界面如下。

![sqli1](/images/sqli1/sqli1-0.png)

首先在 id 参数之后加单引号进行尝试。

![sqli1](/images/sqli1/sqli1-1.png)

出现报错说明有注入点，这里甚至直接看到了 sql 语句。我们用 order by 猜测字段数，order by 2 正常，到 3 之后出现报错，说明有两个字段。

![sqli1](/images/sqli1/sqli1-2.png)

用联合查询查找回显点。

![sqli1](/images/sqli1/sqli1-3.png)

出现了一个回显点。那么我们可以直接爆出所有的库名和表名了。构造如下 poc

    ?id=-1' union select null，group_concat(schema_name) from information_schema.schemata --+

可以看到如下显示。

![sqli1](/images/sqli1/sqli1-4.png)

![sqli1](/images/sqli1/sqli1-5.png)

![sqli1](/images/sqli1/sqli1-6.png)

![sqli1](/images/sqli1/sqli1-7.png)

至此这个靶场的 flag 就拿到了。

