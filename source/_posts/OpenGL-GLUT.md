title: OpenGL-GLUT
date: 2015-12-25 09:39:51
tags:
    - OpenGL
---

# Introduction
　　在以前很多项目使用GLUT库创建OpenGL环境，由于GLUT项目已经放弃开发和维护，在这里建议大家使用新的freeglut库。可以在[freeglut](http://freeglut.sourceforge.net/)网站下载最新的版本。


<!--more-->

# Content
　　在官网上下载对应版本的freeglut，你可以下载源码自己编译或者下载已经编译好的库文件。freeglut的使用基本跟glut的使用一样。在做好前期准备工作后，使用VS创建一个空项目，将freeglut库文件目录添加到项目的库依赖目录。下面介绍怎么使用freeglut创建一个OpenGL环境的窗口：

```

#include <GL/freeglut.h>

//初始化，运行一次
void init()
{
	glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
	glClearDepth(1.0f);
}

//渲染代码，循环运行
void render()
{
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

	glColor3f(1.0f, 0.0f, 0.0f);

	glBegin(GL_TRIANGLES);
	  glVertex3f(-1.0f, -1.0f, 0.0f);
	  glVertex3f(1.0f, 0.0f, 0.0f);
	  glVertex3f(0.0, 1.0, 0.0);
	glEnd();

	glutSwapBuffers();
}

void reshape(int w, int h)
{
	glViewport(0, 0, w, h);
}


int main(int argc, char **argv)
{
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE | GLUT_DEPTH);
	glutInitWindowPosition(300, 0);
	glutInitWindowSize(720, 640);
	glutCreateWindow("Hello World");

	init();
	glutDisplayFunc(render);
	glutReshapeFunc(reshape);
	glutIdleFunc(render);

	glutMainLoop();
}
```
