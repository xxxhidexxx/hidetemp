---
title: "Fiddler 并发方法"
date: 2024-03-16
description: "宝宝辅食"
tags: ["逻辑漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: true
---

并发的应用场景包括但不限于，抽奖，点赞，关注和取关。如果受害者没有主动对频次进行限制，那么这种功能点都是存在并发漏洞的。

fiddler 是好用的抓包工具，和 burp 各有千秋。比 burp 好的地方包括，（1）小程序和手机 app 抓包更流畅；（2）burp 不自带并发，插件的并发功能不是真正的并发，发送频率比较慢，所以我们选择 fiddler 完成任务。这里我们演示一下如何用 fiddler 进行并发。

下载好 Fiddler Classic 之后，首先安装 https 证书，菜单里找到 tools ->> options ->> https 点两个按钮就好了，如下图。

![lo6](/images/weblogic/lo6-1.png)

在浏览器调到 fiddler 默认的 8888 端口。我们打开一个用来测试的页面，下图是 webug 邮件轰炸的靶场。

![lo6](/images/weblogic/lo6-0.png)

在网页端先操作到最后一步，例如填写信息，但是我们先不点击网页的提交按钮，而是回到 fiddler 设置断点。fiddler 也允许对单个 url 设置断点，这里我们为了方便直接设置全局断点，在菜单找到 rules ->> automatic breakpoints ->> before requests 即可设置断点。

![lo6](/images/weblogic/lo6-2.png)

找到我们拦截的请求，鼠标选中，然后在输入法调到英文之后按快捷键 shift+R，可以设置并发的数量。

![lo6](/images/weblogic/lo6-3.png)

![lo6](/images/weblogic/lo6-4.png)

选中并发的所有请求，目前还是拦截，我们点击菜单的 go，发送所有请求，即可完成并发。

![lo6](/images/weblogic/lo6-5.png)