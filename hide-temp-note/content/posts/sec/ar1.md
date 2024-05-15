---
title: "ssrf 任意文件下载靶场演示"
date: 2024-03-15
description: "下载地址所在参数可以访问内网"
tags: ["任意文件漏洞","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

我们用 webug 演示。首先看到一个下载文件的按钮，打开查看器，找到这个按钮，看到路径使用 file 协议，直接访问内网。

![lo1](/images/ar/ar0-0.png)

对 ../ 的数量进行尝试，找到 file=../../../my.ini，点击下载按钮下载 mysql 配置文件。

![lo1](/images/ar/ar0-1.png)