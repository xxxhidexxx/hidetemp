---
title: easy CMS 支付逻辑漏洞"
date: 2024-03-10
description: "改数量为负可实现零元购，前端拦截可绕过"
tags: ["逻辑漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

我们打开这个 easyCMS 的靶场，为了测试支付逻辑，先随便注册一个账户并登录。

打开购买商品的页面，尝试修改数量为负，发现有前端拦截。

![lo2](/images/weblogic/lo2-4.png)

![lo2](/images/weblogic/lo2-5.png)

开 burp 过前端拦截，抓到修改数量的包，将数量改为 -1，发现购买成功。

![lo2](/images/weblogic/lo2-3.png)

![lo2](/images/weblogic/lo2-1.png)

![lo2](/images/weblogic/lo2-2.png)

查看订单，得到 flag=ywag{ijhzxdJHGt}。
