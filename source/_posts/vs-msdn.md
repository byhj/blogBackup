---
title: VS离线安装MSDN
date: 2016-05-16 22:47:21
tags:
 - VS
 
---
# 安装离线MSDN

默认在C:\ProgramData\Microsoft\HelpLibrary2\Catalogs\VisualStudio14\，可以将它移动到你想要的目录下。

然后到网上下载别人下载好的MSDN压缩包，将压缩包解压到前面设置的文档目录，重新打开Help Viewer就会更新内容。


# 创建MSDN Help快捷键

在C:\Program Files <x86>\ Microsoft Help Viewer\v2.2(for VS2015)，找到HlpViewer.exe程序，右键创建快捷键。如果现在
双击它是会报错的，右键属性在目标"C:\Program Files (x86)\Microsoft Help Viewer\v2.2\HlpViewer.exe"后面加上“ /catalogName VisualStudio14 /locale en-us /launchingApp Microsoft, VisualStudio ,14.0”不包括引号。

PS: 如果是其他版本的VS，找到修改对应版本号数据就行了。
