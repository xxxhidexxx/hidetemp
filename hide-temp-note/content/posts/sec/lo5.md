---
title: "熊海 CMS v1.0 越权登录管理员账号"
date: 2024-03-13
description: "添加 cookie 占位即可越权访问后台"
tags: ["逻辑漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

漏洞原理：我们添加一个用来占位的 cookie 保证 cookie 不为空，即可越权访问后台，并且可以避免跳转回登录页面。

这个靶场演示的是熊海 CMS v1.0 的越权漏洞。

尝试将 url 中的 ?r=login 改为 ?r=index 直接进后台，发现无效，会跳回登录页面。

![lo5](/images/weblogic/lo5-0.png)

开 burp 拦截，在 cookie 中添加 user，值可以任意输入，例如将 `Cookie: PHPSESSID=b58boijgnqf8hcg1f62rgb94c3` 改为 `Cookie: PHPSESSID=b58boijgnqf8hcg1f62rgb94c3；user=123` 同时将 url 参数改为 ?r=index。发现直接进入管理员后台。

![lo5](/images/weblogic/lo5-1.png)

![lo5](/images/weblogic/lo5-2.png)

我们刚才是用 burp 抓包修改 cookie，这个 cookie 没有保存，无法真正进入后台页面，随便点一个按钮就跳转回登录页面。因此我们用 cookie-editor 插件添加一个 user 并保存。就可以直接输入 url 越权访问管理员后台，也可以修改管理员账号密码。

![lo5](/images/weblogic/lo5-3.png)

![lo5](/images/weblogic/lo5-4.png)

得到 flag=ywaq{xionghai-czyq}。（其实这个 flag 首页点评论就能看到，不需要操作。）

