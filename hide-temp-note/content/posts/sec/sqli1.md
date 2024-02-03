---
title: "SQL 注入：联合查询"
date: 2024-02-02
description: "靶场演示 mysql，oracle 数据库的联合查询注入方法"
tags: ["SQL 注入", "后端漏洞" ]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

用联合查询进行 sql 注入需要网站有注入点和回显点。有注入点就是说我们塞进去的代码可以被执行，而有回显点意味着我们的注入点所在的语句能够完全控制网页部分内容的显示。此时，通过联合查询可以直接查询出数据库的信息。

