---
title: "YzmCMS 存在反射型 XSS"
date: 2024-03-09
description: "主页面搜索功能三个参数都存在 xss"
tags: ["XSS","前端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

YzmCMS 主页面的搜索功能有三个参数，这三个参数都存在反射性 xss。

![xss1](/images/xss/xss0-0.png)

没有过滤，我们插入常规 payload 即可。

    <img src=a onerror=alert(1)>

![xss1](/images/xss/xss0-1.png)