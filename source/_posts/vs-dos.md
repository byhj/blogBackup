---
title: 为VS Win32程序添加调试窗口
date: 2016-05-16 13:56:50
tags:
   - VS

---

在VS编写Win32程序时, 没有单独的调试窗口，我们可以在main函数的cpp文件中添加：
```
#pragma comment( linker, "/subsystem:\"console\" /entry:\"WinMainCRTStartup\"")
```

如果link无效，替换WinMainCRTStartup
入口函数有：
mainCRTStartup
wmainCRTStartup
wWinMainCRTStartup
WinMainCRTStartup
