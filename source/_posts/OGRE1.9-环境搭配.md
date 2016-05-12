title: OGRE1.9-环境搭配
date: 2015-12-18 22:40:57
tags:
    - OGRE

---
# Introduction
  　　OGRE是一个面向对象的游戏(渲染)引擎，运用了很多常用的设计模式，它的场景管理尤为值得我们学习。官网上虽然提供了SDK开发包，但是如果要深入研究OGRE的架构，使用源码学习是一个更好的选择。本文主要介绍编译OGRE1.9源码的过程，其它版本的环境搭配也类似。
 
# Content

## SDK
　　关于OGRE1.9 SDK的编译，只要下载官网上的安装包，安装对应版本的VS，下载安装Cmake。将OGRE1.9SDK安装目录下CMakeLists.txt文件使用Cmake打开，
选择你想要IDE(在这个选择你的对应VS版本)，点击Configure，然后Generate，编译生成的工程，就能得到Demo合集的程序。你也可以直接使用根目录下得sln
解决方案，用VS打开编译。

  <!--more-->

## Source
下载[OGRE1.9 Source](https://bitbucket.org/sinbad/ogre/downloads)
在OGRE的bitbucket有着它全部的开发信息，在这里你可以下载到那些正在开发版本。在这里我选择Tags下v1-9-0版本，点击右边的zip下载。
下载[Dependencies](https://bitbucket.org/cabalistic/ogredeps/downloads)
同样的选择Tags那里进行下载。使用Cmake生成VS工程，编译生成Dependencies工程，将生成好的库文件目录放到Ogre1.9 Source的根目录下。使用Cmake生成OGRE1.9的VS工程，编译运行。


# Website
- [Building Ogre](http://www.ogre3d.org/tikiwiki/tiki-index.php?page=Building+Ogre)
- [OGRE官网](http://www.ogre3d.org/)
- [OGRE wiki](http://www.ogre3d.org/tikiwiki/tiki-index.php)
