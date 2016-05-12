---
title: 使用VPS搭建Shadowsocks实现代理
date: 2016-05-11 20:20:59
tags:
   - Linux
   - vpn

---

# VPS注册和安装
　　Shadowsocks的正常使用需要服务端，其实所有的翻墙方式都需要服务端，搭建服务端需要你拥有一个属于自己的VPS。VPS提供商有很多，个人推荐[DigitalOcean](https://m.do.co/c/7049364a99a2), 注册和安装非常容易，可以绑定国内的银行卡。

# Shadowsocks安装

首先根据当前系统安装Shadowsocks:

Debian / Ubuntu:

```
apt-get install python-pip
pip install shadowsocks
```

<!--more-->

CentOS:
```
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

然后可以直接在后台运行：
```
ssserver -p 8000 -k password -m aes-256-cfb -d start
```

当然也可以使用配置文件进行配置，方法创建/etc/shadowsocks.json文件
```
vi /etc/shadowsocks.json
```
　　填入如下内容,其中server_ip是你vps分配的ip， server_port可以随便设置一个没有占用的，local_adresss和local_port下面的默认值，password密码自己定义，method为加密方式，这些参数在客户端需要对应才能使用：
```
{
    "server":"server_ip",
    "server_port":8000,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-cfb"
}
```
然后使用配置文件在后台运行：
```
sudo ssserver -c /etc/shadowsocks.json -d start
```

停止运行：
```
sudo ssserver -c /etc/shadowsocks.json -d stop
```

如果要检查日志：
```
sudo less /var/log/shadowsocks.log
```

# 使用　　
　　到[Shadowsocks](https://shadowsocks.org/en/download/clients.html)下载对应的客户端，在github的[shadowsocks](https://shadowsocks.org/en/download/clients.html)项目有着源码和使用说明，如果你想进一步了解Shadowsocks情况可以自己到
这个github中阅读有关信息。安装好客户端后，点击列表中的服务器，新加一个代理服务器，其中的参数就是上面设置的参数，然后选择系统代理就可以在浏览器中访问国外的网站。
你也可以自定义使用代理的网站白名单。

　　需要注意的是Shadowsocks一般只能通过浏览器进行翻墙，并不是真正的全局vpn，如果需要使得本机程序也使用代理的话，可以使用一个叫Proxifier的软件进行设置，具体可以参看我另一篇教程:[使用Proxifier使得Shadowsocks本机程序进行代理]()。
