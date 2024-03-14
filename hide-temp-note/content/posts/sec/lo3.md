---
title: "vl CMS 无密码登录"
date: 2024-03-11
description: "无密码登录其他人的账号"
tags: ["逻辑漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

打开靶场页面如下。

![lo3](/images/weblogic/lo3-1.png)

利用 nday 已知信息，将 url 修改为 index.php?s=/member/res_login/ 并抓包。

将 GET 改为 POST 并在结尾加上 id=1。

![lo3](/images/weblogic/lo3-2.png)

回到主页刷新，发现已经登录到其他人的账号。

![lo3](/images/weblogic/lo3-3.png)

进入个人中心得到 flag=ywaq{uhgzxdygfh}。

添加 &id=1 是常用绕过手法。