title: QT-static
date: 2016-01-22 10:26:45
tags: Qt静态编译-MSVC
---
# Introduction

# Content
安装python, perl, ruby. 将这些bin目录设置到path中，打开对应版本的开发人员命令提示， cd到Qt目录执行环境设置。
打开解压目录-qtbase-mkspecs-win32-msvc2013(选择你的编译器版本)
修改qmake.cof文件内的参数设置：
```
QMAKE_CFLAGS_RELEASE    = -O2 -MD -Zc:strictStrings
QMAKE_CFLAGS_RELEASE_WITH_DEBUGINFO += -O2 -MD -Zi -Zc:strictStrings
QMAKE_CFLAGS_DEBUG      = -Zi -MDd
//将以上改为
QMAKE_CFLAGS_RELEASE    = -O2 -MT -Zc:strictStrings
QMAKE_CFLAGS_RELEASE_WITH_DEBUGINFO += -O2 -MT -Zi -Zc:strictStrings
QMAKE_CFLAGS_DEBUG      = -Zi -MTd
```
<!--more-->

回到解压目录，打开VS开发人员命令窗口 C:\Program Files (x86)\Microsoft Visual Studio 12.0\Developer Command Prompt for VS2013", 首先执行下面参数：

```
configure -confirm-license -opensource -mp -platform win32-msvc2013 -debug-and-release -static -prefix " D:\Qt\5.5.0-static-vs2013" -qt-sql-sqlite -qt-sql-odbc -plugin-sql-sqlite -plugin-sql-odbc -qt-zlib -qt-libpng -qt-libjpeg -opengl desktop -qt-freetype -no-qml-debug -no-angle -nomake tests -nomake examples -skip qtwebkit
```

接下来执行nmake，等很久会编译完毕，然后nmake install进行安装

```
nmake
nmake install
```
