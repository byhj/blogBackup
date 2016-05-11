---
title: 使用Proxifier使得Shadowsocks本机程序进行代理
date: 2016-05-11 21:06:52
tags:
   - shadowsocks
   - vpn
---

# 介绍：

Shadowsocks:

　　shadowsocks一个可穿透防火墙的轻量代理，目前已经支持包括WIN，安卓，IOS等所有平台。Shadowsocks的安装可以参考我的另一篇文章[使用VPS搭建Shadowsocks代理]();
Proxifier:

　　[Proxifier](http://www.proxifier.com/download.htm)是一款功能非常强大的socks5客户端，可以让不支持通过代理服务器工作的网络程序能通过HTTPS或SOCKS代理或代理链。支持 64位系统，支持Xp，Vista，Win7，MAC OS ,支持socks4，socks5，http代理协议，支持TCP，UDP协议，可以指定端口，指定IP，指定域名,指定程序等运行模式，兼容性非常好，和SOCKSCAP属于同类软件，不过SOCKSCAP已经很久没更新了，不支持64位系统。 有许多网络应用程序不支持通过代理服务器工作，Proxifier 解决了这些问题和所有限制，让您有机会不受任何限制使用你喜爱的软件。此外，它让你获得了额外的网络安全控制，创建代理隧道，并添加使用更多网络功能的权力。

<!--more-->

# 使用

运行Shadowsocks客户端：

<img width = "400" src= "http://7xtwmz.com1.z0.glb.clouddn.com/shadowsocks.jpg">

运行Proxifier，点击配置文件-代理服务器-添加 地址和端口输入本地的参数，类型选SOCKS5，这样运行时会使得本机所有网络活动通过代理。

<img width = "400" src= "http://7xtwmz.com1.z0.glb.clouddn.com/pro.jpg">

你也可以通过代理规则指定特定程序进行代理，加速代理。其中Direct是不使用代理的意思，Proxy就是通过Shadowsocks进行代理，除此之外也可以通过右键的菜单栏
Proxifer选择打开要进行代理的程序。

<img width = "400" src= "http://7xtwmz.com1.z0.glb.clouddn.com/pro2.jpg">
