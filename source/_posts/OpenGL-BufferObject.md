title: OpenGL-BufferObject
date: 2015-09-20 22:40:46
tags: OpenGL

---
# Introdution
Buffer Objects是存储OpenGL Context(GPU)分配的一组非格式化内存的OpenGL Objects. 这些内存能够用来存储顶点数据，图像或者帧缓存返回的像素数据和其它数据。
 
# Creation
Buffer Objects是OpenGL Objects,所以它们遵循所有OpenGL Objects的规定。你能使用glGenBuffersc创建Buffer Objects, 使用glDeleteBuffers删除它们。这些都是大多数OpenGL Object使用的标准生成和创建Objects的模式。
``` 
void glBindBuffer(enum target, uint bufferName);
```
Buffer Objects存储的是任意大小的线性内存数组，这些内存在使用前需要被分配，有两种为Buffer Objects 分配内存的方法：可变或者不可变。为buffer分配不可变内存会改变你跟Buffer Object交互的方式。

<!--more-->

## Immutable Storage
Buffer Objects被分配为不可变方式时，你不能为这个Buffer重新分配内存。使用explicit invalidation
command 或者mapping the buffer.
``` 
void glBufferStorage(GLenum target, GLsizeiptr size, const GLvoid * data, GLbitfield flags);
```
### Immutable access methods
- Writing to the buffer with any rendering pipeline process. These include Transform Feedback, Image Load Store, Atomic Counter, and Shader Storage Buffer Object. Basically, anything that is part of the rendering pipeline that can write to a buffer will always work.

- Clearing the buffer. Because this only transfers a few bytes of data, it is not considered "client-side" modification.

- Copying the buffer. This copies from one buffer to another, so it all happens "server-side".
Invalidating the buffer. This only wipes out the contents of the buffer, so it is considered "server-side".

- Asynchronous pixel transfers into the buffer. This sets the data in a buffer, but only through pure-OpenGL mechanisms.

- Using glGetBufferSubData? to read a part of the buffer back to the CPU. This is not a "server-side" operation, but it's always available regardless.

---

# Data Specification
因为glBufferData在分配内存空间同时更新数据，所以并不适合更新那些已经分配内存的数据（这对于immutable storage buffers是不可能的）

我们可以使用下面的函数去更新buffer部分的数据：
``` 
void glBufferSubData(enum target, intptr offset, sizeiptr size,  const void *data)
```
## Clearing
Buffer objects 内存能够被部分或全部清除为指定的值。这些函数的工作原理与Pixel Transfer操作类似。
``` 
void glClearBufferData(GLenum target, GLenum internalformat, GLenum format, GLenum type, const void * data);
void glClearBufferSubData(GLenum target, GLenum internalformat, GLintptr offset,  GLsizeiptr size, GLenum format, GLenum type, const void * data);
```
一个清除全部内容，一个清除部分内容

## Copying
一个buffer的数据可以被复制到另一个buffer，我们为源和目标buffer绑定不同的target, 一般使用GL_COPY_READ_BUFFER GL_COPY_WRITE_BUFFER作为相应的标志。
```
void glCopyBufferSubData​(GLenum readtarget​, GLenum writetarget​, GLintptr readoffset​,GLintptr writeoffset​, GLsizeiptr size​);
```
readtarget是绑定到source buffer上的buffer, 这是复制数据的提供者，writetarget 是绑定到destination buffer的buffer，是复制数据的接受者。

## Mapping
对于一般情况来说，glBufferSubData是一种很好的更新数据方式。但有时取决你的使用方式，它的性能会减低。例如，你的算法生成了一些将要存储至buffer object的数据，你首先必须为这些数据分配暂时的存储区，然后使用glBufferSubData函数复制到OpenGL管理的内存。当你想要将数据写回时，glGetBufferSubData也许并不是你想要的方式， 虽然它看起来功能相像。如果你能得到一个指向buffer object存储区的指针，然后直接写入，这样的方式会非常好。
为了达到上面所说的效果，你必须对buffer进行映射以得到一个指向可读写buffer内存的指针。
```
void *glMapBufferRange​(GLenum target​, GLintptr offset​, GLsizeiptr length​,GLbitfield access​);
```

---
# General use
- GL_ARRAY_BUFFER
- GL_ELEMENT_ARRAY_BUFFER
- GL_COPY_READ_BUFFER & GL_COPY_WRITE_BUFFER
- GL_PIXEL_UNPACK_BUFFER & GL_PIXEL_PACK_BUFFER
- GL_QUERY_BUFFER
- GL_TEXTURE_BUFFER
- GL_TRANSFORM_FEEDBACK_BUFFER
- GL_UNIFORM_BUFFER
- GL_DRAW_INDIRECT_BUFFER
- GL_ATOMIC_COUNTER_BUFFER
- GL_DISPATCH_INDIRECT_BUFFER
- GL_SHADER_STORAGE_BUFFER

##Binding indexed targets
```
 void glBindBufferRange​(GLenum target​, GLuint index​, GLuint buffer​,  GLintptr offset​, GLsizeiptr size​ );
```

## Multibind and indexed targets
```
void glBindBuffersRange​(GLenum target​, GLuint first​, GLsizei count​, const GLuint *buffers​, const GLintptr *offsets​, const GLintptr *sizes​);
```

---
# Reference


