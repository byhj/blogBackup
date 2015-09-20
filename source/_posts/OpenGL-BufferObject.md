title: OpenGL-BufferObject
date: 2015-09-20 22:40:46
tags: OpenGL

---
# Introdution
Buffer Objects �Ǵ洢OpenGL Context(GPU)�����һ��Ǹ�ʽ���ڴ��OpenGL Objects. ��Щ�ڴ��ܹ������洢�������ݣ�ͼ�����֡���淵�ص��������ݺ��������ݡ�
 
# Creation
Buffer Objects ��OpenGL Objects,����������ѭ����OpenGL Objects�Ĺ涨������ʹ��glGenBuffersc����Buffer Objects, ʹ��glDeleteBuffersɾ�����ǡ���Щ���Ǵ����OpenGL Objectʹ�õı�׼���ɺʹ���Objects��ģʽ��
``` 
void glBindBuffer(enum target, uint bufferName);
```
Buffer Objects�洢���������С�������ڴ����飬��Щ�ڴ���ʹ��ǰ��Ҫ�����䣬������ΪBuffer Objects �����ڴ�ķ������ɱ���߲��ɱ䡣Ϊbuffer���䲻�ɱ��ڴ��ı����Buffer Object�����ķ�ʽ��

## Immutable Storage
Buffer Objects������Ϊ���ɱ䷽ʽʱ���㲻��Ϊ���Buffer���·����ڴ档ʹ��explicit invalidation
command ����mapping the buffer.
``` 
void glBufferStorage?(GLenum target?, GLsizeiptr size?, 
                     const GLvoid * data?, GLbitfield flags?);
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
��ΪglBufferData���������������ݣ�Ҳ���Է����ڴ�ռ䣬���Բ����ʺϸ�����Щ�Ѿ������ڴ�����ݣ������immutable storage buffers�ǲ����ܵģ�

���ǿ���ʹ������ĺ���ȥ����buffer���ֵ����ݣ�
``` 
void glBufferSubData?(enum target, intptr offset, sizeiptr size,  const void *data)
```
## Clearing
Buffer objects �ڴ��ܹ������ֻ�ȫ�����Ϊָ����ֵ����Щ�����Ĺ���ԭ����Pixel Transfer�������ơ�
``` 
void glClearBufferData?(GLenum target??, GLenum internalformat?, GLenum format?, 
                       GLenum type?, const void * data?);
void glClearBufferSubData?(GLenum target?, GLenum internalformat?, GLintptr offset?, 
                          GLsizeiptr size?, GLenum format?, GLenum type?, const void * data?);
```
## Copying
һ��buffer�����ݿ��Ա����Ƶ���һ��buffer������ΪԴ��Ŀ��buffer�󶨲�ͬ��target, һ��ʹ��GL_COPY_READ_BUFFER GL_COPY_WRITE_BUFFER

---
# General use

---
# Reference


