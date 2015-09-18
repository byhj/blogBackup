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
# Contents


---
#   
