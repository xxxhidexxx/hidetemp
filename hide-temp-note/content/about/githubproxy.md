---
title: "和 GitHub 相关的网络代理设置"
date: 2024-01-29
description: "让 powershell，ssh，git 走代理节点"
tags: [网络设置]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: misc
comment: false
draft: false
---

我们的浏览器通过 http 代理可以正常访问 github，现在我们希望通过 ssh 远程关联 github 仓库。

遇到了两个问题：1，默认的端口 22 连接超时；2，改成端口 443 依然连接超时。

首先，在打开 powershell 终端的时候，设置

    set http_proxy=127.0.0.1：10809
    set https_proxy=127.0.0.1：10809

记得别打空格。这样就设置好了 power shell 的代理。

其次，在 `C:/Users/someone/.ssh` 打开 git bash 输入 `touch config` 建立一个 config 文件，并在 config 中写入 

    Host github.com
    Hostname ssh.github.com
    Port 443
    User git

这一步解决的是用 ssh 连接 github 走 http 代理。

最后再来设置 git 的代理。终端输入 

    git config --global http.proxy 127.0.0.1:10809
