title: OpenGL-Objects
date: 2015-09-17 22:05:15
tags: OpenGL

---
# Introduction
 　　在OpenGL中，OpenGL Objects是一种包含着一些状态的结构。当你把OpenGL Objects绑定到上下文环境时，Objects内含的状态就会映射到上下文环境中的状态。在这个时候，如果你通过API函数修改了上下文中的状态，对应OpenGL Ojects内的状态也会被修改。
　　 OpenGL被定义为一种“状态机”，它所提供的多种API函数调用可以改变OpenGL的状态，也可以查询某些状态量，或者使用当前的状态去Rendering。
　　 OpenGL有着多种类型的Ojbects, 它们存储着不同的状态，一组状态被封装在一个OpenGL Objects中，我们可以通过函数调用去改变它们。
　　 需要注意的是，OpenGL只是制定相应的规范，具体实现是由各个显卡驱动的厂商决定的，但实现也一般都会符合OpenGL制定的规范。

---

<!--more-->

# Creation & Destruction

``` 
void glGen*(GLsizei n, GLuint *objects)
```
 　　该函数创建n个对象名称(整数值引用)，返回指向这些objects的指针。在这一步，并不需要创建objects的状态数据。对于大多数object来说，在第一次绑定到上下文时，只包含其默认状态。在绑定前使用Objects是非法的。需要注意的是，Program Pipeline Ojbects和 Sampler Objects并不遵循这种运行机制。
 　　Object名称类型是GLuint, 它是一个数值型引用，用来识别一个object, 其中数值0在不同的Object中有着特定意义。
  （在OpenGL3.0之前，可以完全忽略上述的的分配函数,直接绑定认为合法的Ojbect名称）
``` 
void glDelete*(GLsizei n​, const GLuint *objects​);
```
 　　当我们使用完Objects，可以使用上面函数进行删除。任何非法Object和Object 0会被忽略。Objects被删除后，它们的名字可以通过glGen*重新分配。
 
## Deletion unbinding
　　当绑定到当前context的object被删除时，会解除所有该object的绑定。
（Binding goes to the context; attachment is what you do when one object references another. Attachments are not severed due to this call.）
　　当一个object attached到container object，而该object绑定到context，删除会解除attach操作，如果该object没有绑定到context, attach操作不会被破坏。

Some objects can be bound to the context in unusual ways. These ways include, but are not limited to:
- Buffer Objects bound to indexed targets via glBindBufferRange​ or its equivalents.
- Textures bound as images.

## Deletion orphaning
　　调用glDelete”并不保证马上消除object的内容和它的名字，如果一个object在删除后还被使用，那么当前object名字还是合法的。

An object is "in use" if:
- It is bound to a context. This is not necessarily the current one, since deleting it will automatically unbind it from the context that caused the deletion. Though remember the caveat above about non-standard binding points for GL 4.4 and before.
- It is attached to a container object.

---
#   Usage
　　由于Ojbects是一组状态，当要修改Ojbects前，需要把它绑定到OpenGL context。在绑定后，调用API函数可以对context中的状态进行改变，Objects的相应状态也会被改变。在绑定一个新生成的Object名称时，会为这个Object产生新的状态

```
void glBind*(GLenum target​, GLuint object​);
```
## Object zero
 　　数值0在一些OpenGL Objects中有着特定的含义。在大多数情况下，object 0就像是空指针NULL，是一种指示作用。对于一些Objects来说，0代表“default object", Texture 和 Framebuffers都有着这样的概念。

---
# Multibind
　　在OpenGL4.4引入了多重绑定概念，可以一次把不同类型的Objects绑定到各自的目标上，这种方式只是用来绑定后使用而不是修改。在多重绑定中，任何一个绑定错误都会导致整体的错误。

---
# Sharing
　　你可以创建多个OpenGL contexts, 而当前GL context是线程相关的，一般来说它们各自是完成独立开来的，不会相互影响。但是context创建的时候，你可以使用另一个已经存在context的Objects. 需要注意，那些包含着其它Objects引用的Objects不能被共享，剩下的Objects可以进行context间共享，包括GLSL Objects和Sync Objects这种不遵循OpenGL Object机制的Objects.
　　一个context中Objects状态的改变并不是在另一个context马上可见，在这里有着特定的规则保证object状态数据的可见性。如果你使用线程，你需要自己做一些同步处理工作以保证在一个context使用那些会改变的objects之前，objects的改变在另一个context中已经完成。

---
# Types
　　Objects can be separated into two different categories: regular objects and container objects. Here is the list of regular objects.

**Regular objects**
- Buffer Objects
- Query Objects
- Renderbuffer Objects
- Sampler Objects
- Texture Objects

**Container objects**
- Framebuffer Objects
- Program Pipeline Objects
- Transform Feedback Objects
- Vertex Array Objects

---
# Names
　　正如上面所说，OpenGL Ojbects的名字是一个由系统分配的整数值，这种方式对于用户并不直观，在调试时也有很大的缺陷。在OpenGL 4.3 Core引入了一种为Object作标志的方法。
``` 
void glObjectLabel​(GLenum identifier​, GLuint name​, 
                   GLsizei length​, const char * label​);
                   
void glObjectPtrLabel​(void * ptr​, GLsizei length​, const char * label​);
```
第一种方法可以为所有的Objects设置Label，第二种为Sync Objects设置Label

| Identifier    | Object Type  |
| ------------- |:-------------:| 
| GL_BUFFER	    | Buffer Object | 
| GL_SHADER     | 	Shader Object | 
| GL_PROGRAM    | 	Program Object | 
| GL_VERTEX_ARRAY	 | Vertex Array Object | 
| GL_QUERY	 | Query Object | 
| GL_PROGRAM_PIPELINE | 	Program Pipeline Object | 
| GL_TRANSFORM_FEEDBACK	 | Transform Feedback Object | 
| GL_SAMPLER	 | Sampler Object | 
| GL_TEXTURE	 | Texture Object | 
| GL_RENDERBUFFER	 | Renderbuffer Object | 
| GL_FRAMEBUFFER	 | Framebuffer Object | 

查询设置Label的Objects
```
void glGetObjectLabel​(GLenum identifier​, GLuint name​, GLsizei bufSize​,
                      GLsizei * length​, char * label​);

void glGetObjectPtrLabel​(void * ptr​, GLsizei bufSize​, 
                         GLsizei * length​, char * label​);
```

---
# Other
　　The following are "objects", but they do not follow the standard conventions laid out on this page for OpenGL objects:
- Sync Objects
- Shader and Program Objects

Except for program pipeline objects, which do follow the OpenGL Object conventions.

[--Original: OpenGL Wiki--](https://www.opengl.org/wiki/OpenGL_Object)