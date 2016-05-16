---
title: Ubuntu 16.04编译安装安装Shadowsocks-qt5
date: 2016-05-16 09:46:24
tags:
   - Ubuntu
   - VPN

---

# 介绍
正常的安装方法通过PPA：

```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```
但是不知为什么运行一会儿就程序崩溃了，经过一番折腾都不行。一种方法是使用[Shadowsocks Pyton版]
```
apt-get install python-pip
pip install shadowsocks
```
我试了之后没有反应，所以就自己编译源码了。到Qt官网下载Qt5+安装，然后在安装Shadowsocks依赖包：
```
sudo apt-get install qt5-qmake qtbase5-dev libqrencode-dev libqtshadowsocks-dev libappindicator-dev libzbar-dev libbotan1.10-dev
```

<!--more-->
使用Qt Creater打开[Shadowsocks-qt5](https://github.com/shadowsocks/shadowsocks-qt5)源码包中的.pro工程，编译成功运行，然后添加你的服务器的信息就可以了。

浏览器中使用SwitchyOmega进行代理设置：
<img width = "80%" src= "http://7xtwmz.com1.z0.glb.clouddn.com/ss-so.jpg">
