title: QT-static
date: 2016-01-22 10:26:45
tags: Qt静态编译-MSVC
---
# Introduction

# Content
安装python, perl, ruby.打开对应版本的开发人员命令提示， cd到Qt目录执行环境设置。

设置：
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

参数：

```
configure -confirm-license -opensource -mp -platform win32-msvc2013 -debug-and-release -static -prefix " C:\Qt\5.5.0-static-vs2013" -qt-sql-sqlite -qt-sql-odbc -plugin-sql-sqlite -plugin-sql-odbc -qt-zlib -qt-libpng -qt-libjpeg -opengl desktop -qt-freetype -no-qml-debug -no-angle -nomake tests -nomake examples -skip qtwebkit
```

```
nmake
nmake install
```
