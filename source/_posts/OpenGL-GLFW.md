title: OpenGL-GLFW
date: 2015-12-25 09:39:32
tags:
   - OpenGL
---

# Introduction
　　GLFW是一个创建OpenGL环境窗口的开源跨平台程序库，它同时可以处理输入和事件。目前来说，它比OpenGL以前使用的GLUT框架更加成熟和方便。更多信息在[GLFW](http://www.glfw.org/)的官网有介绍。本文主要简单介绍GLFW的使用。

# Content
　　在官网上下载对应版本的GLFW，你可以下载源码自己编译或者下载已经编译好的库文件。由于最新的GLFW3跟以前的GLFW版本有所区别，所以从旧版本升级的程序需要按照官网提供的教程进行修改.在做好前期准备工作后，使用VS创建一个空项目，将glfw库文件目录添加到项目的库依赖目录。下面介绍怎么使用GLFW创建一个OpenGL环境的窗口：

<!--more-->
```

#include <GLFW/glfw3.h>
#include <iostream>

int main(int argc, char **argv)
{

  //初始化GLFW环境
	if (!glfwInit())
	{
		std::cerr << "Error: can not init the glfw!" << std::endl;
		return -1;
	}

	//创建一个窗口化的OpenGL环境窗口
	GLFWwindow *pWindow = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
	if (!pWindow)
	{
		glfwTerminate();
		return -1;
	}

	//使用创建的OpenGL环境窗口
	glfwMakeContextCurrent(pWindow);

	//循环直到用户关闭窗口
	while (!glfwWindowShouldClose(pWindow))
	{
		//在这里添加渲染代码

		//交换双缓存
		glfwSwapBuffers(pWindow);

		//处理事件
		glfwPollEvents();
	}

  //结束，销毁初始化创建的资源
	glfwTerminate();

	return 0;
}

```
