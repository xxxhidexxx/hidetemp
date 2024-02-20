---
title: "用 Burp Suite 跑 MySQL 盲注"
date: 2024-02-07
description: "用 burp 跑脚本很方便，但是时间盲注最好手工"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: true
---

## 提要

盲注就是存在注入点但是看不到回显点。这通常有两种情况，或者是输入 `?id=1' and 1=2 --+` 能看到报错页面，或者是输入 `?id=1' and sleep(3) --+` 回车能看到网页延迟刷新。这里 sleep(3) 延迟的不一定是三秒也可能是 3 的倍数，因为 sleep 可能被执行了不止一次。数据库一般都有延时函数，例如 mysql 的 sleep()，这种注入方法一般称为时间盲注。在页面没有任何明确显示的情况下，把响应时间作为一种显示，这个想法很有趣，然而，在现实场景中时间盲注成本比较高，运算量可能比较大，不是很稳定，除非一定要爆破，应该谨慎考虑这个选择。

我们这里主要演示的是用 burp 脚本进行爆破。如果手工注入的话语句的 = 改成 > 然后使用二分法就可以了，也没什么好讲的。

在做这个靶场的时候我们发现 burp 跑出来的时间盲注结果和自己手工测试的不一样。相同的测试语句，手工的结果是正确的，但是 burp 的响应时间比较抽象，还不知道原因。

用 burp 跑有报错的布尔盲注倒是很顺利，如下所述。

## 演示

还是用之前的靶场进行测试。首先用 `' and 1=2 --+` 找到注入点。看到页面报错。

![sqli3](/images/sqli1/sqli3-4.png)

打开 burp 抓包后发送到 intruder，选择集束炸弹。选中要遍历的参数之后点击添加 payload。我们两个的 payload 点位分别是 ascii 和字符串位数，所以类型都选择数值，ascii 我们直接从 1 跑到 128。

![sqli3](/images/sqli1/sqli3-0.png)

点击 attack 按钮之后可以看到响应包的长度有不同，比较长的就说明是正常显示的。我们把不报错的 ascii 码依次解码得到数据库名 webug。

![sqli3](/images/sqli1/sqli3-1.png)

然后可以进一步爆破表名和字段名找到 flag。语句的写法例如

    ' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>1 --+

讲一下 sql 语法。这里的 substr 的用法是 substr(object,start,length) 所以 start 遍历 1~n，然后 length 永远写 1。这里 limit 0,1 的意思是从第 0 项开始查询 1 项，所以是第一个参数遍历 0~n，第二个参数恒为 1。

后面重复劳动不是很重要懒得写了。

然后我们用时间盲注跑一下看看，得到下述结果，响应时间对不上，原因未知，应该是 burp 的问题。

我们的语句用手工测试的结果没有问题，语法肯定是对的。也讲一下这个语法，if(e1,e2,e3) 的意思是，如果 e1 是对的就执行 e2，否则执行 e3。

![sqli3](/images/sqli1/sqli3-5.png)

![sqli3](/images/sqli1/sqli3-6.png)
