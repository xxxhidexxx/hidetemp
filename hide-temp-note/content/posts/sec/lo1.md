---
title: "大米 CMS 支付逻辑漏洞"
date: 2024-03-10
description: "修改前端参数可实现零元购，可以修改账户余额"
tags: ["逻辑漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

打开大米 CMS 的靶场，找到支付页面。提示必须登录才可以继续下一步，我们随便注册一个账户即可。

![lo1](/images/weblogic/lo1-1.png)

首先发现这个数量可以直接修改成 -1，但是提交无效。

![lo1](/images/weblogic/lo1-4.png)

我们还是用正常的购买流程，开 burp 拦截。修改参数 qty=-1，代表数量变为 -1，提交之后购买成功。

可以看出这里直接将前端传递的参数作为金额和账户余额进行结算。因此我们将订单的产品数量修改为负数可以实现账户充值。

![lo1](/images/weblogic/lo1-2.png)

![lo1](/images/weblogic/lo1-3.png)

![lo1](/images/weblogic/lo1-7.png)

订单页面看到了 flag=ywaq{zxdijGcztU}。
