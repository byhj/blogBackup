title: 在GLFW3中使用AntTweakBar
date: 2015-12-25 10:27:26
tags:
    - OpenGL
    - GUI

---
# Introduction
　　本文主要介绍怎么在GLFW3中使用AntTweakBar吗，旧版本的GLFW使用在AntTweakBar的Example中已经有了，所以在这里不作介绍。注意，程序是在OpenGL-Framework里面介绍的代码为基础的，如果有不了解的地方，请先阅读那一篇教程.

<!--more-->

# Content

App.h

```
#ifndef __APP_H_
#define __APP_H_

#include <GL/glew.h>
#include <GLFW/glfw3.h>

#include <iostream>
#include <string>
#include <windows.h>
#include <memory>
#include <AntTweakBar.h>

#define USE_ANT

#ifdef _WIN32
const int ScreenWidth = static_cast<int>( GetSystemMetrics(SM_CXSCREEN) * 0.75 );
const int ScreenHeight = static_cast<int>(  GetSystemMetrics(SM_CYSCREEN) * 0.75 );
const int PosX = (GetSystemMetrics(SM_CXSCREEN) - ScreenWidth) ;
const int PosY = (GetSystemMetrics(SM_CYSCREEN) - ScreenHeight);
#else
const int ScreenWidth = 1200;
const int ScreenHeight = 1000;
const int PosX = 300;
const int PosY = 100;
#endif

namespace byhj
{

namespace ogl
{

	class App
	{
	public:
		App() {}
		virtual ~App() {}

	public:
		void Run(std::shared_ptr<App> the_app);

		//Override
		virtual void v_InitInfo() = 0;
		virtual void v_Init()	  = 0;
		virtual void v_Update()   = 0;
		virtual void v_Render()	  = 0;
		virtual void v_Shutdown() = 0;

		virtual void v_KeyCallback(GLFWwindow* window, int key, int scancode, int action, int mode);
		virtual void v_Movement(GLFWwindow *window) {}
		virtual void v_MouseCallback(GLFWwindow* window, double xpos, double ypos) {}
		virtual void v_ScrollCallback(GLFWwindow* window, double xoffset, double yoffset) {}

	protected:
		struct WindowInfo
		{
			WindowInfo() :title("OpenGL BlueBook : "),
				Width(ScreenWidth),
				Height(ScreenHeight),
				posX(PosX),
				posY(PosY) {}

			std::string title;
			int Width;
			int Height;
			int posX, posY;
		}windowInfo;

		float GetAspect();
		int GetScreenWidth();
		int GetScreenHeight();

		std::string GetGLRenderer()
		{
			return m_GLRenderer;
		}
		std::string GetGLVersion()
		{
			return m_GLVersion;
		}
		std::string GetGLSLVersion()
		{
			return m_GLSLVersion;
		}
	private:
		static  std::shared_ptr<App> app;

		static void glfw_key(GLFWwindow * window, int key, int scancode, int action, int mode);
		static void glfw_mouse(GLFWwindow* window, double xpos, double ypos);
		static void glfw_scroll(GLFWwindow* window, double xoffset, double yoffset);
		static void glfw_mouseButton(GLFWwindow *window, int x, int y, int z);
		static void glfw_char(GLFWwindow *window, unsigned int x);

		std::string m_GLRenderer;
		std::string m_GLVersion;
		std::string m_GLSLVersion;

	};  //class

}  //namespace


} //namespace

#endif  //
```

App.cpp

```
#include "App.h"


namespace byhj
{
namespace ogl
{

////////////////////////////////////////////////////////////////////////////////////////

void App::v_KeyCallback(GLFWwindow* window, int key, int scancode, int action, int mode)
{
	if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
		glfwSetWindowShouldClose(window, GL_TRUE);
	if (key == GLFW_KEY_C && action == GLFW_PRESS)
		glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_NORMAL);
}

std::shared_ptr<App> App::app;

////////////////////////////////////////////////////////////////////////////////////////

 void App::glfw_key(GLFWwindow * window, int key, int scancode, int action, int mode)
{
	if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
		glfwSetWindowShouldClose(window, GL_TRUE);

#ifdef USE_ANT
	TwEventKeyGLFW(key, action);

	app->v_KeyCallback(window, key, scancode, action, mode);
#endif
}

 ////////////////////////////////////////////////////////////////////////////////////////

void App::glfw_mouse(GLFWwindow* window, double xpos, double ypos)
{
#ifdef USE_ANT
	TwEventMousePosGLFW(xpos, ypos);

	app->v_MouseCallback(window, xpos, ypos);
#endif
}

////////////////////////////////////////////////////////////////////////////////////////

 void App::glfw_scroll(GLFWwindow* window, double xoffset, double yoffset)
{
#ifdef USE_ANT
	TwEventMouseWheelGLFW(xoffset);

	app->v_ScrollCallback(window, xoffset, yoffset);
#endif
}

 ////////////////////////////////////////////////////////////////////////////////////////
void App::glfw_mouseButton(GLFWwindow *window, int x, int y, int z)
{
	TwEventMouseButtonGLFW(x, y);
}
 void App::glfw_char(GLFWwindow *window, unsigned int x)
{
	TwEventCharGLFW(x, GLFW_PRESS);
}

 ////////////////////////////////////////////////////////////////////////////////////////


void App::Run(std::shared_ptr<App> the_app)
{
	app = the_app;

	std::cout << "Starting GLFW context" << std::endl;
	if (!glfwInit())
	{
		std::cerr << "Failed to initialize GLFW" << std::endl;
		return;
	}

	v_InitInfo();

	//Add this to use the debug output
	glfwWindowHint( GLFW_OPENGL_DEBUG_CONTEXT, true );


	GLFWwindow *window = glfwCreateWindow(windowInfo.Width, windowInfo.Height,
		                                  windowInfo.title.c_str(), nullptr, nullptr);
	glfwSetWindowPos(window, windowInfo.posX, windowInfo.posY);
	glfwMakeContextCurrent(window);


	glfwSetCursorPosCallback(window, glfw_mouse);          // - Directly redirect GLFW mouse position events to AntTweakBar
	glfwSetScrollCallback(window, glfw_scroll);    // - Directly redirect GLFW mouse wheel events to AntTweakBar
	glfwSetKeyCallback(window, glfw_key);                         // - Directly redirect GLFW key events to AntTweakBar
#ifdef USE_ANT
	glfwSetMouseButtonCallback(window, glfw_mouseButton); // - Directly redirect GLFW mouse button events to AntTweakBar
	glfwSetCharCallback(window, glfw_char);                      // - Directly redirect GLFW char events to AntTweakBar
#endif
	//glfwSetInputMode(window, GLFW_STICKY_KEYS, GL_TRUE);
	// GLFW Options
	glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);

	if (window == NULL)
	{
		std::cerr << "Failed to create GLFW window" << std::endl;
		glfwTerminate();
		return ;
	}
	glewExperimental = GL_TRUE;

	//Check the GLSL and OpenGL status
	if (glewInit() != GLEW_OK)
	{
		std::cerr << "Failed to initialize GLEW" << std::endl;
		return ;
	}
	const GLubyte *renderer = glGetString( GL_RENDERER );  
	const GLubyte *vendor = glGetString( GL_VENDOR );  
	const GLubyte *version = glGetString( GL_VERSION );  
	const GLubyte *glslVersion = glGetString( GL_SHADING_LANGUAGE_VERSION );  

	m_GLRenderer = (const char *)renderer;
	m_GLVersion  = (const char *)version;
	m_GLSLVersion = (const char *)glslVersion;

	GLint major, minor;  
	glGetIntegerv(GL_MAJOR_VERSION, &major);  
	glGetIntegerv(GL_MINOR_VERSION, &minor);  
	std::cout << "GL Vendor    :" << vendor << std::endl;  
	std::cout << "GL Renderer  : " << renderer << std::endl;  
	std::cout << "GL Version (std::string)  : " << version << std::endl;  
	std::cout << "GL Version (integer) : " << major << "." << minor << std::endl;  
	std::cout << "GLSL Version : " << glslVersion << std::endl;    
	std::cout << "--------------------------------------------------------------------------------"
		      << std::endl;
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, major); //opengl 4.3
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, minor);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
	glfwWindowHint(GLFW_RESIZABLE, GL_TRUE);

	// Create a GLFWwindow object that we can use for GLFW's functions
	v_Init();



	while (!glfwWindowShouldClose(window))
	{
		int sw, sh;
		glfwGetWindowSize(window, &sw, &sh);

		glViewport(0, 0, sw, sh);

		glfwPollEvents();
		v_Movement(window);

		v_Update();
		//Render for the object
		v_Render();

		glfwSwapBuffers(window);
	}
	v_Shutdown();


	glfwTerminate();
}

float App::GetAspect()
{
	return static_cast<float>(ScreenWidth) / static_cast<float>(ScreenHeight);
}

int App::GetScreenWidth()
{
	return ScreenWidth;
}

int App::GetScreenHeight()
{
	return ScreenHeight;
}

}

}

```

使用AntTweakBar
