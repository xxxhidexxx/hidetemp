---
title: "Kali Linux 网络设置"
date: 2024-02-10
description: "连接和代理"
tags: ["网络设置"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "sec"
comment: false
draft: false
---

连不上网的时候可以用如下指令。

    dhclient -v

![kaliifconfig](/images/linux/kali_ip.png)

代理走本机的话可以用本机开 clash。

创建脚本。

    #!/bin/bash
    # encoding: utf-8

    Proxy_IP=192.168.1.1
    Proxy_Port=8080

    # Set System Proxy
    function xyon(){
        export https_proxy=http://$Proxy_IP:$Proxy_Port
        export http_proxy=http://$Proxy_IP:$Proxy_Port
        export all_proxy=socks5://$Proxy_IP:$Proxy_Port
        echo -e "System Proxy is $Proxy_IP:$Proxy_Port"
    }

    # unSet System Proxy
    function xyoff(){
        unset all_proxy
        unset https_proxy
        unset http_proxy
        echo -e "System Proxy is Disabled"
    }

    # Default Function is Set Proxy
    if [ $# != 0 ]
    then
	    if [ $1 == 'off' ]
	    then
		    xyoff
	    elif [ $1 == 'on' ]
	    then
		    xyon
	    else
		    echo "Please Input on or off!"
	    fi
    else
	    echo "Please input command."
    fi

调用脚本。

    chmod +x setproxy.sh
    
    source setproxy.sh on
    
    source setproxy.sh off