---
title: extern "C"用法解析
date: 2016-05-13 16:22:36
tags:
  - C
  - C++
---

# extern "C"
包含双重含义：首先它修饰的目标是“extern”的；其次被它修饰的目标是“C”的。

---
# extern

extern是C/C++语言中表明函数和全局变量作用范围（可见性）的关键字，该关键字告诉编译器，其声明的函数和变量可以在本模块或其它模块中使用。
```
extern int a;
```
　　上面是一个变量的声明而不是定义，编译器并没有为变量a分配内存空间。extern表示这是一个外部变量，该变量的定义在其它地方(文件)，C++规定变量只能定义一次。在模块的头文件中对本模块提供给其它模块引用的函数和全局变量，通常会以关键字extern修饰。
　　例如，如果模块B欲引用该模块A中定义的全局变量和函数时只需包含模块A的头文件即可。这样，模块B中调用模块A中的函数时，在编译阶段，模块B虽然找不到该函数，但是并不会报错；它会在连接阶段中从模块A编译生成的目标代码中找到此函数。与extern对应的关键字是static，被它修饰的全局变量和函数只能在本模块中使用。

<!--more-->

---

# "C"
　　被extern "C"修饰的变量和函数按照C语言方式编译和连接，首先C++支持函数重载，而C不支持。当函数使用C++编译后在符号库中的名字与C语言的不同。C++会对函数重载进行不同的命名符号的修改，看下面的例子：
```
void foo( int x, int y);
```
　　该函数被C编译器编译后在符号库中的名字为_foo，而C++编译器则会产生像_foo_int_int之类的名字（不同的编译器可能生成的名字不同，但是都采用了相同的机制，这种方法通常被称为“mangled name”。
　　例子中的_foo_int_int这样的名字包含了函数名、函数参数数量及类型信息，C++就是靠这种机制来实现函数重载的。例如，在C++中，函数void foo( int x, int y )与void foo( int x, float y )编译生成的符号是不相同的，后者为_foo_int_float。
```
#ifndef Test_H
#define Test_H
extern "C" int foo( int x, int y );
#endif
```
上面的extern "c"代码告诉编译器你希望使用C方式去编译该函数，既不支持函数重载

---

# 惯用法_cplusplus

```
#ifdef __cplusplus
extern "C " {
#endif

///在这里声明或其它

#ifdef __cplusplus
}
#endif
```
　　上面的方法的意思是:代码中的__cplusplus是cpp中的自定义宏，如果编译器检查定义了这个宏的话表示这是一段cpp的代码。也就是说，上面的代码的含义是:如果这是一段cpp的代码，那么加入"extern "C"{" 和 " }"处理其中的代码，其中{ }内部的代码是通过extern"C"进行处理，既使用C方式进行处理。
