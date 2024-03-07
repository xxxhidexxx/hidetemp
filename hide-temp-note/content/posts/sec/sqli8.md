---
title: "掌控安全靶场 pass-15"
date: 2024-03-06
description: "宽字节注入"
tags: ["SQL 注入","后端漏洞"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

起因。网站开发防止 SQL 注入添加了 PHP 函数：addslashes()。php语言中的addslashes()函数在指定的预定义字符前添加反斜杠，这些字符是单引号(')，双引号(")，反斜线（\）与NUL（NULL字符）。例如客户端提交的参数中如果含有单引号，双引号等这些特殊字符，addslashes 函数则会在单引号前加反斜线“\”，将单引号转义成没有功能性的字符。

构造 payload：
    %df' or 1=1 -- qwe
    %df' or 1=0 -- qwe

则 %df\' 被十六进制编码变为 %df%5c%27，而 df5c 被 gbk 编码变为汉字“运”，从而单引号 %27 被解析。

绕过这个斜杠之后，这个靶场剩下的部分是走流程用 union 读取 flag，此处从略。

![sqli8](/images/sqli1/sqli8-0.png)
