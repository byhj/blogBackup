title: Cocos2dx-AndroidStudio
date: 2015-12-16 21:26:37
tags:
      - Cocos2dx
      - Android

---

# 1.简介
   本文主要简单介绍Cocos2dx中怎么使用AndroidStudio开发.

# 2.测试环境
Windows10
Cocos2dx3.9
AndroidStudio1.3.2

# 3.内容

   Cocos2dx的环境配置安装官网上说的进行，下载所需要的sdk，在Cocos2dx 的build文件夹中用VS2015打开解决方案，编译链接程序。编译成功后使用Cocos控制台命令创建一个HelloWorld工程.

 <!--more-->
```
cocos new HellWorld -p com.byhj -l cpp -d D:/Cocos2dX
```
  在D盘的Cocos2dx文件夹中可以看见新生成的HelloWorld的cocos2dx工程，里面有着多个平台开发的项目选择，一般我们使用VS编写调试工程，最后移植到所需要的平台。接着就是使用AndroidStudio测试的关键部分，因为AndroidStudio使用gradle构建系统，所以具体NDK的使用跟Eclipse有所区别，而且在AndroisStudio1.3以后，使用gradle-experimental支持NDK开发。在新生成HelloWorld工程根目录打开cmd，输入下面命令：
```
cocos run/compile -p android --android-studio      
```
 这个命令使用NDK将C++文件生成Android平台下的so库文件，通过JNI被Java程序调用，具体细节暂时不用去探究，当NDK编译完成后，使用AndroidStudio导入HelloWorld工程里面的proj.android-studio工程。  编译链接，正常情况下可以使用手机设备或者安卓模拟器看到HelloWorld的测试场景。
