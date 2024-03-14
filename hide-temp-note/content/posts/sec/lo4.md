---
title: "MetInfo v4.0 越权修改密码"
date: 2024-03-13
description: "修改密码的包替换 userid 可以修改任意用户密码，之后可以登录到任意用户"
tags: ["逻辑漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

这个靶场演示的是 MetInfo v4.0 的 nday。

先注册一个账号，找到修改密码，然后抓包。

![lo4](/images/weblogic/lo4-0.png)

![lo4](/images/weblogic/lo4-1.png)

将 userid 改为任意用户，密码自己设置。

![lo4](/images/weblogic/lo4-2.png)

用其他人的用户名和自己设置的密码，成功登录到其他人的账号。

![lo4](/images/weblogic/lo4-3.png)

进入个人中心找到 flag=ywaq{uhgJHzxdIG}。