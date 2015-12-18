title: OpenGL-HelloWorld
date: 2015-12-18 13:58:49
tags:
     - OpenGL

---

# Introduction
　　本文将介绍怎么使用OpenGL3+编写一个Triangle,使用了glfw3创建窗口渲染环境。
在你自己进行编写时，根据你自己的环境下载下面两个对应版本的库文件，如果使用VS作为IDE,记得将库文件目录添加到VS的工程库文件依赖的目录
- [glfw](http://www.glfw.org/)
- [glew](http://glew.sourceforge.net/)

<!--more-->

# Content
首先我们创建一个渲染框架，这在以后编写其它类似的OpenGL程序时，可以简化很多重复的工作。

**APP.h**

```
#ifndef __APP_H_
#define __APP_H_

#include <GL/glew.h>
#include <GLFW/glfw3.h>

#include <iostream>
#include <string>
#include <windows.h>
#include <memory>

//如果是在Window环境下，我们使用WindowAPI将渲染窗口定位在PC屏幕的正中间
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

	protected:

  //窗口信息结构体
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

	private:
		static  std::shared_ptr<App> app;

	};  //class

}  //namespace

} //namespace

#endif  //
```

**APP.cpp**
```
#include "App.h"

namespace byhj
{
namespace ogl
{

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
	std::cout << "--------------------------------------------------------------------------------" << std::endl;

	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, major);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, minor);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
	glfwWindowHint(GLFW_RESIZABLE, GL_TRUE);

	// Create a GLFWwindow object that we can use for GLFW's functions
	v_Init();

   //这里是主渲染循环
	while (!glfwWindowShouldClose(window))
	{
		int sw, sh;
		glfwGetWindowSize(window, &sw, &sh);
		glViewport(0, 0, sw, sh);

		glfwPollEvents();
		v_Movement(window);

		v_Update();
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
　由于OpenGL可编程流水线要求我们自己编写Shader文件且与程序进行装配，所以我实现了一个装载Shader文件的小模块。

**Shader.h**

```
#ifndef Shader_H
#define Shader_H

#include <GL/glew.h>
#include <iostream>
#include <vector>
#include <string>
#include <windows.h>

#ifdef _WIN32
#define WINDOW_PLATFORM
#endif

namespace byhj
{

   namespace ogl
   {
   char *textFileRead(const char *fn);

    class Shader
    {

    public:
			Shader() = default;
      Shader(std::string shaderName) : m_Name(shaderName) {}
			~Shader() = default;

        public:
        	void init(std::string folder = "");
        	void attach(int type, char *filename);
        	void link();
        	void info();
        	void use()  const;
        	void end()  const;

        	GLuint GetProgram() const;

        private:
			    GLuint m_Program = 0;
			    std::string fileDir;
        	std::string m_Name = "Shader";   
        };
    }
}
#endif
```
**Shader.cpp**

```
#include "Shader.h"
#include <fstream>

namespace byhj
{

namespace ogl
{

//read the Shadercode
char * textFileRead(const char *fn) {  //read the shader code
	FILE *fp;  
	char *content = NULL;  
	int count=0;  

	if (fn != NULL) {  

		fopen_s(&fp , fn, "rt");
		if (!fp){

#ifdef WINDOW_PLATFORM
			MessageBox(NULL, "Can not open the shader file", "Error",  MB_OK | MB_ICONERROR | MB_TASKMODAL);
#else
			std::cerr << "Can not open the shader file:" << fn << std::endl;

#endif
			return NULL;
		}
		if (fp != NULL) {  
			fseek(fp, 0, SEEK_END);  
			count = ftell(fp);  
			rewind(fp);  

			if (count > 0) {  
				content = (char *)malloc(sizeof(char) * (count+1));  
				count = fread(content,sizeof(char),count,fp);  
				content[count] = '\0';  
			}  
			fclose(fp);  
		}
		else
			std::cout << "Fail to open the shader file" << std::endl;
	}  
	return content;  
}  

void Shader::init(std::string folder)
{
	fileDir = folder;
	m_Program = glCreateProgram();
}

void Shader::attach(int type, char* filename) //连接不同种类Shader
{
	std::string file = fileDir + filename;
	auto mem = textFileRead(file.c_str());

	GLuint handle = glCreateShader(type);
	glShaderSource(handle, 1, (const GLchar**)(&mem), 0);

	glCompileShader(handle);

	GLint compileSuccess=0;
	GLchar compilerSpew[256];

	glGetShaderiv(handle, GL_COMPILE_STATUS, &compileSuccess);

	if (!compileSuccess)
	{
		glGetShaderInfoLog(handle, sizeof(compilerSpew), 0, compilerSpew);
		printf("Shader%s\n%s\ncompileSuccess=%d\n",filename, compilerSpew, compileSuccess);

		while(1);;
	}
	glAttachShader(m_Program, handle);

}

void Shader::link()
{
	glLinkProgram(m_Program);  

	GLint linkSuccess;
	GLchar compilerSpew[256];
	glGetProgramiv(m_Program, GL_LINK_STATUS, &linkSuccess); //print the shader info
	if(!linkSuccess)
	{
		glGetProgramInfoLog(m_Program, sizeof(compilerSpew), 0, compilerSpew);
		printf("ShaderLinker:\n%s\nlinkSuccess=%d\n",compilerSpew,linkSuccess);
		while(1);;
	}

	std::cout << m_Name << " linked successful" << std::endl;
	std::cout << "--------------------------------------------------------------------------------" << std::endl;
}

void Shader::info()
{
	std::cout << "------------------------------" << m_Name << " Interface-------------------------"
		      << std::endl;

	GLint outputs = 0;
	glGetProgramInterfaceiv(m_Program, GL_PROGRAM_INPUT,  GL_ACTIVE_RESOURCES, &outputs);
	static const GLenum props[] = {GL_TYPE, GL_LOCATION};
	GLint params[2];
	GLchar name[64];
	const char *type_name;

	if (outputs > 0)
		std::cout << "***Input***" << std::endl;

	for (int i = 0; i != outputs; ++i)
	{
		glGetProgramResourceName(m_Program, GL_PROGRAM_INPUT, i, sizeof(name), NULL, name);
		glGetProgramResourceiv(m_Program, GL_PROGRAM_INPUT, i, 2, props, 2, NULL, params);
		type_name = name;
		//std::cout << "Index " << i << std::endl;
		std::cout <<  "(" <<  type_name  << ")" << " location: " << params[1] << std::endl;
	}

	glGetProgramInterfaceiv(m_Program, GL_PROGRAM_OUTPUT,  GL_ACTIVE_RESOURCES, &outputs);
	if (outputs > 0)
		std::cout << "***Onput***" << std::endl;

	for (int i = 0; i != outputs; ++i)
	{
		glGetProgramResourceName(m_Program, GL_PROGRAM_OUTPUT, i, sizeof(name), NULL, name);
		glGetProgramResourceiv(  m_Program, GL_PROGRAM_OUTPUT, i, 2, props, 2, NULL, params);

		type_name = name;
		//std::cout << "Index " << i << std::endl;
		std::cout  <<  "(" <<  type_name  << ")" << " location: " << params[1] << std::endl;
	}

	glGetProgramInterfaceiv(m_Program, GL_UNIFORM_BLOCK,  GL_ACTIVE_RESOURCES, &outputs);
	if (outputs > 0)
		std::cout << "***Uniform Block***" << std::endl;

	for (int i = 0; i != outputs; ++i)
	{
		glGetProgramResourceName(m_Program, GL_UNIFORM_BLOCK, i, sizeof(name), NULL, name);
		glGetProgramResourceiv(  m_Program, GL_UNIFORM_BLOCK, i, 2, props, 2, NULL, params);

		type_name = name;
		//std::cout << "Index " << i << std::endl;
		std::cout  <<  "(" <<  type_name  << ")" << " location: " << params[1] << std::endl;
	}

	glGetProgramInterfaceiv(m_Program, GL_UNIFORM,  GL_ACTIVE_RESOURCES, &outputs);
	if (outputs > 0)
		std::cout << "***Uniform***" << std::endl;
	if (outputs > 100)
		return ;
	for (int i = 0; i != outputs; ++i)
	{
		glGetProgramResourceName(m_Program, GL_UNIFORM, i, sizeof(name), NULL, name);
		glGetProgramResourceiv(  m_Program, GL_UNIFORM, i, 2, props, 2, NULL, params);

		type_name = name;
		//std::cout << "Index " << i << std::endl;
		std::cout  <<  "(" <<  type_name  << ")" << " location: " << params[1] << std::endl;
	}
	std::cout << "--------------------------------------------------------------------------------" << std::endl;
}

void Shader::use() const
{
	glUseProgram(m_Program);
}

void Shader::end() const
{
	glUseProgram(0);
}

GLuint Shader::GetProgram() const
{
	return m_Program;
}

}
}
```

下面是Triangle渲染部分，主要是初始化顶点数据，加载Shader，然后进行渲染.

**Triangle.h**

```
#ifndef Triangle_H
#define Triangle_H

#include <GL/glew.h>
#include "ogl/Shader.h"

namespace byhj
{

class Triangle
{
public:
	Triangle()  = default;
	~Triangle() = default;

	void Init();
	void Render();
	void Shutdown();

private:
	void init_buffer();
	void init_vertexArray();
	void init_shader();

	ogl::Shader TriangleShader{ "Triangle Shader" };
	GLuint program = 0;
};

}
#endif

```

**Triangle.cpp**
```

#include "Triangle.h"

namespace byhj
{

void Triangle::Init()
{
	init_buffer();
	init_vertexArray();
	init_shader();
}

void Triangle::Render()
{
	//Use this shader and vao data to render
	glUseProgram(program);

	glDrawArrays(GL_TRIANGLES, 0, 3);

	glUseProgram(0);
}


void Triangle::Shutdown()
{
	glDeleteProgram(program);
}

void Triangle::init_buffer()
{
}

void Triangle::init_vertexArray()
{
}

void Triangle::init_shader()
{
	TriangleShader.init();
	TriangleShader.attach(GL_VERTEX_SHADER, "triangle.vert");
	TriangleShader.attach(GL_FRAGMENT_SHADER, "triangle.frag");
	TriangleShader.link();
	program = TriangleShader.GetProgram();
}

}
```

**RenderSystem.h**

```
#ifndef RENDERSYSTEM_H
#define RENDERSYSTEM_H

#include "App.h"
#include "Triangle.h"

namespace byhj
{

class RenderSystem : public ogl::App
{
public:
	RenderSystem();
	~RenderSystem();

public:
	void v_InitInfo();
	void v_Init();
	void v_Update();
	void v_Render();
	void v_Shutdown();

private:
	byhj::Triangle m_Triangle;
};

}

#endif
```
**RenderSystem.cpp**

```
#include <GL/glew.h>
#include "RenderSystem.h"

namespace byhj
{

RenderSystem::RenderSystem()
{

}

RenderSystem::~RenderSystem()
{

}

void RenderSystem::v_InitInfo()
{
	windowInfo.title += "Triangle";
}

void RenderSystem::v_Init()
{
	m_Triangle.Init();
}

void RenderSystem::v_Update()
{
}

void RenderSystem::v_Render()
{

	static const GLfloat black[] = { 0.0f, 0.0f, 0.0f, 1.0f };
	glClearBufferfv(GL_COLOR, 0, &black[0]);

	m_Triangle.Render();
}

void RenderSystem::v_Shutdown()
{
	m_Triangle.Shutdown();
}

}
```

**main.cpp**

```

#include "RenderSystem.h"
#include <memory>

int main(int argc, const char **argv)
{
	auto app = std::make_shared<byhj::RenderSystem>();
	app->Run(app);

	return 0;
}

```

---
